---
title: Serverless Architecture with Terraform - REST API
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2025-04-01 11:33:00 +0900
categories: [DevOps, Terraform]
tags: [aws, terraform, serverless, iac, lambda]
pin: false
math: false
---


## 들어가면서

회사의 비지니스적인 상황에 따라서, Serverless Architecture(AWS Lambda + API gateway)를 사용할때가 있습니다.    
Serverless 환경에서 개발할때에 여러 어려움중에 하나가, 개발 Cycle에 대한 불편함인데,  
Local에서 HTTP server를 구동하는게 어려워서, AWS Lambda에 배포하여 테스트하는 경우가 많다는 겁니다.  
역시, 이런 어려움은 저만 겪는게 아니라, 이미 Solution이 있습니다. 바로 ‘[Serverless Framework](https://www.serverless.com/)’입니다.

### [Serverless Framework](https://www.serverless.com/)의 아쉬운점

‘Serverless Framework’는 ‘Serverless-Offline’모듈을 통해, Local환경에서 Lambda기반의 HTTP Server를 가상화해서 활용할 수 있습니다.  
또한, ‘Serverless Framework‘로 자동화된 배포까지 구현할 수 있습니다.  
즉, ‘개발환경’과 ‘배포환경’을 하나의 Framework로 다룰 수 있습니다.  
  
‘Serverless Framework'는 AWS의 CloudFormation 기반으로 작동합니다. 때문에, Cloudformation기반으로 인프라의 State를 자동으로 맞춰주고, Rollback까지 손쉽게 진행할 수 있습니다.  
하지만, **많은 회사에서 이미 ‘Terraform’을 IaC(Infrastructure as Code)도구로 사용할텐데, 별도의 IaC Stack을 추가하는건 피하게 됩니다**.    
  
  
때문에, ‘Terraform을 이용해서, Serverless framework와 같이 구현할 수 있지 않을까?’라는 생각으로 이 포스팅을 진행하게 되었습니다.

## 본문

먼저 작업을 완료한 Repository를 공유합니다. 이 포스팅과 함께 내용을 따라오시면, 이해에 도움이 되실겁니다.  
[terraform-aws-examples/examples/lambda-base/apigateway at main · KanghoonYi/terraform-aws-examples](https://github.com/KanghoonYi/terraform-aws-examples/tree/main/examples/lambda-base/apigateway)
_(Terraform 기반의 serverless architecture example)_

### 접근 방법 및 과정

Terraform자체는 IaC도구로서, Application Code를 handling하는 기능은 없습니다.  

때문에, 다음과 같은 기능을 직접 구현해야 합니다.  
: Source Code를 Lambda용으로 build (외부모듈 dependency를 포함하여 zip파일로 생성)
: Source Code기반으로 HTTP server를 Local환경에 가상화

이를 통해, ‘AWS Lambda’를 생성하는 과정을 자동화 하게 됩니다.  
이후, API Gateway에 연결하기 위해 이 설정에 ‘API Gateway’에 대한 dependency를 설정하게 됩니다.  
  
  
이렇게 해서 완료된 환경을 AWS의 [Guide](https://aws.amazon.com/ko/blogs/compute/better-together-aws-sam-cli-and-hashicorp-terraform/)에 따라, ‘AWS SAM’으로 Local에서 HTTP server로 가상화하여 구동하게 됩니다.  

### Terraform의 Module기능 활용

Terraform환경에서 Lambda를 쉽게 활용하기 위한 별도의 [Module](https://registry.terraform.io/modules/terraform-aws-modules/lambda/aws/latest)이 준비되어 있지만, 이 Module은 여전히  API Gateway와의 Dependency나 Execution Role과 같이 설정할 것들이 많습니다.  
이를 해소하고자, [terraform-aws-modules/lambda/aws](https://registry.terraform.io/modules/terraform-aws-modules/lambda/aws/latest) 모듈 기반의 Custom 모듈을 정의해서 사용합니다.  
[terraform-aws-examples/examples/lambda-base/apigateway/tfModules/APIGatewayHandler at main · KanghoonYi/terraform-aws-examples](https://github.com/KanghoonYi/terraform-aws-examples/tree/main/examples/lambda-base/apigateway/tfModules/APIGatewayHandler)

### Source Code Build

[https://github.com/KanghoonYi/terraform-aws-examples/blob/main/examples/lambda-base/apigateway/package.json](https://github.com/KanghoonYi/terraform-aws-examples/blob/main/examples/lambda-base/apigateway/package.json)

```json
# examples/lambda-base/apigateway/package.json
{
  "name": "apigateway",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build:dev": "NODE_ENV=dev tsx esbuild.ts && (cd .esbuild && npm install --omit=dev)",
    "build:prod": "NODE_ENV=production tsx esbuild.ts && (cd .esbuild && npm install --omit=dev)",
    "local-run:dev": "sam local start-api --hook-name terraform --beta-features"
  },
  ...
}
```

[https://github.com/KanghoonYi/terraform-aws-examples/blob/main/examples/lambda-base/apigateway/esbuild.ts](https://github.com/KanghoonYi/terraform-aws-examples/blob/main/examples/lambda-base/apigateway/esbuild.ts)

```tsx
// examples/lambda-base/apigateway/esbuild.ts

import { globSync } from 'glob';
import esbuildModule from 'esbuild';
import { packageJsonPlugin } from 'esbuild-plugin-package-json';
import * as process from 'node:process';

const entryFilePaths = globSync('src/functions/**/handler.ts', {
  // root: path.resolve(__dirname, 'src/functions'),
  nodir: true,
  dotRelative: true,
});

(async () => {
  const buildResult = await esbuildModule.build({
    plugins: [packageJsonPlugin()],
    entryPoints: entryFilePaths,
    entryNames: '[dir]/[name]',
    outbase: './',
    outdir: '.esbuild',
    bundle: true,
    platform: 'node',
    tsconfig: './tsconfig.json',
    treeShaking: true,
    packages: 'external',
    sourcemap: true,
    minify: process.env.NODE_ENV === 'production',
    format: 'esm',
  });
  return buildResult;
})().catch((reason) => {
  console.error(reason);
  process.exit(1);
});

```

`package.json` 의 ‘script’를 이용하여, nodejs기반의 Lambda source code를 build하는 과정을 명시합니다.  
`NODE_ENV=dev tsx esbuild.ts && (cd .esbuild && npm install --omit=dev)`  
1. `NODE_ENV=dev tsx esbuild.ts`  
   ‘tsx’를 이용해서 source code를 build하는것을 말합니다.

2. `cd .esbuild && npm install --omit=dev`  
   build된 결과물을 기준으로, 외부 dependency를 다시 설치합니다.


### Souce Code build 과정을 Terraform에 표현하기
여기서는 Terraform에서 Source code build 과정을 명시해주고, Local 구동이 가능하게 ‘AWS SAM Metdata’를 자동으로 생성하게 해줘야 합니다.

[https://github.com/KanghoonYi/terraform-aws-examples/blob/main/examples/lambda-base/apigateway/tfModules/APIGatewayHandler/main.tf](https://github.com/KanghoonYi/terraform-aws-examples/blob/main/examples/lambda-base/apigateway/tfModules/APIGatewayHandler/main.tf)

```bash
# examples/lambda-base/apigateway/tfModules/APIGatewayHandler/main.tf

locals {
  esbuild_src   = "./.esbuild"
  function_name = "${var.lambda_handler_name}-${var.stage}"
  default_env = {
    STAGE = var.stage
  }
  combined_env = merge(local.default_env, var.environment_variables)
}

# Terraform 사용시 Source code를 새롭게 build하도록 설정
resource "terraform_data" "build_app" {
  triggers_replace = {
    always_run = timestamp()
  }
  provisioner "local-exec" {
    command = "rm -rf ${local.esbuild_src} && export PATH=$PATH:${path.cwd}/node_modules/.bin && npm run build:prod"
  }
}

# AWS SAM을 위한 metadata 생성
resource "null_resource" "sam_metadata_aws_lambda_function" {
  count = 1

  provisioner "local-exec" {
    command = "rm -rf ${local.esbuild_src} && export PATH=$PATH:${path.cwd}/node_modules/.bin && npm run build:prod"
  }

  triggers = {
    # This is a way to let SAM CLI correlates between the Lambda function resource, and this metadata
    # resource
    resource_name = "module.APIGatewayLambdaHandler.aws_lambda_function.this[0]"
    resource_type = "ZIP_LAMBDA_FUNCTION"

    # The Lambda function source code.
    # original_source_code = jsonencode(var.source_path)
    original_source_code = local.esbuild_src

    # a property to let SAM CLI knows where to find the Lambda function source code if the provided
    # value for original_source_code attribute is map.
    # source_code_property = "path"

    # A property to let SAM CLI knows where to find the Lambda function built output
    built_output_path = "${local.esbuild_src}/${module.APIGatewayLambdaHandler.local_filename == null ? "" : module.APIGatewayLambdaHandler.local_filename == null}"
  }

  # SAM CLI can run terraform apply -target metadata resource, and this will apply the building
  # resources as well
  depends_on = [terraform_data.build_app, module.APIGatewayLambdaHandler]
}
```

  

### Terraform에서 Source Code에 대한 Lambda 정의하기
Terraform으로 명시한 ‘Source Code Build’ 결과물을 ZIP파일로 만들고, Lambda로 명시합니다.

[https://github.com/KanghoonYi/terraform-aws-examples/blob/main/examples/lambda-base/apigateway/tfModules/APIGatewayHandler/main.tf](https://github.com/KanghoonYi/terraform-aws-examples/blob/main/examples/lambda-base/apigateway/tfModules/APIGatewayHandler/main.tf)  

```bash
module "APIGatewayLambdaHandler" {
  source        = "terraform-aws-modules/lambda/aws"
  architectures = ["arm64"]
  timeout       = 15

  function_name       = local.function_name
  description         = var.lambda_description
  handler             = var.handler_src
  runtime             = "nodejs22.x"
  memory_size         = 256
  publish             = true
  create_function     = true
  create_package      = true
  create_role         = false
  create_sam_metadata = false # 상위 과정에서 sam metadata를 수동으로 생성해야 하기 때문에, false로 세팅합니다

  source_path = [
    {
      path = local.esbuild_src
      commands = [
        "npm install --production",
        ":zip" # AWS Lambda에 적용하기 위해, source code를 zip파일로 압축합니다.
      ],
      patterns : [
        "node_modules/.+",
        "!node_modules/@aws-sdk/.*",
      ]
    }
  ]

  store_on_s3 = true
  s3_bucket   = var.source_code_bucket_name

  environment_variables = local.combined_env == null ? {} : local.combined_env

  lambda_role = var.lambda_role_arn

  event_source_mapping = {
	  # SQS와 같이 Event mapping기능을 사용해야하는 경우에 여기에서 설정하게 됩니다.
  }

  tracing_mode          = "Active"
  attach_network_policy = true
  attach_tracing_policy = true

  logging_log_group                 = "/aws/lambda/${local.function_name}" # Cloudwatch에서 사용할 Log Group이름을 설정합니다
  cloudwatch_logs_retention_in_days = var.log_ttl_days

  depends_on = [
    terraform_data.build_app,
  ]
}

# Proxy Resource (/{proxy+})
resource "aws_api_gateway_resource" "proxy" {
  rest_api_id = var.agw_rest_api_id
  parent_id   = var.agw_parent_resource_id
  path_part   = var.agw_http_path
}

# Method (ANY)
resource "aws_api_gateway_method" "proxy_method" {
  rest_api_id   = var.agw_rest_api_id
  resource_id   = aws_api_gateway_resource.proxy.id
  http_method   = var.agw_http_method
  authorization = "NONE"
}

# Integration with Lambda
resource "aws_api_gateway_integration" "lambda" {
  rest_api_id             = var.agw_rest_api_id
  resource_id             = aws_api_gateway_resource.proxy.id
  http_method             = aws_api_gateway_method.proxy_method.http_method
  integration_http_method = "POST" # API Gateway에서 Lambda를 실행할때 사용하는 method입니다. Lambda를 Invoke하는 것은 'POST'로만 가능합니다.
  type                    = "AWS_PROXY"
  uri                     = module.APIGatewayLambdaHandler.lambda_function_invoke_arn
}

# Lambda Permission for API Gateway
# API Gateway에서 Lambda를 실행할 수 있는 권한을 명시합니다.
resource "aws_lambda_permission" "apigw" {
  statement_id  = "AllowAPIGatewayInvoke"
  action        = "lambda:InvokeFunction"
  function_name = module.APIGatewayLambdaHandler.lambda_function_name
  principal     = "apigateway.amazonaws.com"
  source_arn    = "${var.agw_execution_arn}/*/*"
}
```



### Local 에서 HTTP Server 실행 방법

이제 AWS SAM을 이용해서, Terraform환경의 세팅을 기반으로 HTTP server를 실행해 보겠습니다.  

> AWS SAM은 Docker기반으로 실행됩니다. 때문에, Docker Daemon이 실행중이어야 합니다.
{: .prompt-info }

> AWS SAM은 AWS CLI를 기반으로 작동하기 때문에, AWS Token관련 환경변수가 설정되어 있어야 합니다.
{: .prompt-info }

```json
# examples/lambda-base/apigateway/package.json

{
  "name": "apigateway",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "build:dev": "NODE_ENV=dev tsx esbuild.ts && (cd .esbuild && npm install --omit=dev)",
    "build:prod": "NODE_ENV=production tsx esbuild.ts && (cd .esbuild && npm install --omit=dev)",
    "local-run:dev": "sam local start-api --hook-name terraform --beta-features"
  },
  "devDependencies": {
    ...
  }
}
```

위의 `package.json` 내용중, 아래와 같은 실행 명령어를 사용합니다.

```bash
$ NODE_ENV=dev tsx esbuild.ts && (cd .esbuild && npm install --omit=dev) ## 혹은 npm run buidl:dev
$ sam local start-api --hook-name terraform --beta-features 
```

그러면, 아래와 같은 메세지가 뜹니다.

```bash
Experimental features are enabled for this session.                                                                                                                                         
Visit the docs page to learn more about the AWS Beta terms https://aws.amazon.com/service-terms/.                                                                                           
                                                                                                                                                                                            
Mounting APIGatewayGetHello-dev at http://127.0.0.1:3000/hello [GET]                                                                                                                        
You can now browse to the above endpoints to invoke your functions. You do not need to restart/reload SAM CLI while working on your functions, changes will be reflected                    
instantly/automatically. If you used sam build before running local commands, you will need to re-run sam build for the changes to be picked up. You only need to restart SAM CLI if you    
update your AWS SAM template                                                                                                                                                                
2025-04-03 20:57:59 WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:3000
2025-04-03 20:57:59 Press CTRL+C to quit

```

여기서 `http://127.0.0.1:3000/hello [GET]` 부분을 확인할 수 있습니다.  
이제 이 URL을 통해 HTTP 요청을 보내서 Application을 테스트해 볼 수 있습니다.  

### Source Code 배포 방법

Source Code를 배포할 때는, terraform을 적용할때와 같습니다.

```bash
$ terraform apply -var-file=./vars/dev.tfvars
```

## 한계

이런식으로 Terraform환경에서도 Serverless Architecture를 구현할 수 있지만, Local구동시에도 Docker와 같은 각종 의존성이 생기는 문제가 있습니다.  
또한 이로 인해(Docker기반으로 Local서버가 실행되어서), Debugger를 이용해서 Application을 debuging하는 환경은 아직 세팅되어 있지 않습니다.

## References

Terraform Module 구조
: [Standard Module Structure \| Terraform \| HashiCorp Developer](https://developer.hashicorp.com/terraform/language/modules/develop/structure)

Terraform에서 사용하는 AWS Spec 확인하기
: [Terraform Registry](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

terraform-aws-module과 AWS SAM 연동
: [Terraform Registry](https://registry.terraform.io/modules/terraform-aws-modules/lambda/aws/latest#sam_cli_integration)
: [Better together: AWS SAM CLI and HashiCorp Terraform \| Amazon Web Services](https://aws.amazon.com/ko/blogs/compute/better-together-aws-sam-cli-and-hashicorp-terraform/)

Terraform관련 AWS 공식 문서
: [AWS SAM CLI Terraform support - AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/terraform-support.html)

AWS SAM을 이용해서 local에서 test하기
: [Using the AWS SAM CLI with Terraform for local debugging and testing - AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-samcli-terraform.html)
