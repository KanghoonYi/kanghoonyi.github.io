---
title: V8 engine build and check machine code(V8 engine을 build하고 machine code 확인하기)
author: KanghoonYi(pour)
name: KanghoonYi(pour)
date: 2024-07-05 14:10:02 +0900
categories: [Typescript, Javascript]
tags: [javascript, machine code, cpu, nodejs, v8]
pin: false
math: false
---

Google의 V8엔진을 통해, javascript는 새로운 기회를 얻었습니다.  
다른 script언어들과 다르게, javascript코드를 compile하여 machine code로 cache함으로써 성능을 극한으로 끌어 올렸습니다.  
문득, V8이 생성하는 ‘Machine code’라는게 궁금해졌습니다.  
그래서 직접 Machine code를 확인하고자 합니다.

## What is ‘Machine code(기계어)’?

‘Machine code’는 binary형식의 데이터이긴 하지만, CPU가 직접 읽고 실행할 수 있는 데이터를 말합니다.  
우리에게 익숙한 각종 파일들이 binary데이터 형식이긴 하지만, CPU가 직접 읽을 수 있는 명령들은 아닙니다.  
반면에, ‘Machine code’는 CPU가 읽을 수 있는 CPU의 API로 이루어진 데이터입니다. 이에 따라, CPU Architecture에 따라 ‘Machine code’는 바뀌게 됩니다.

## V8에서 생성하는 ‘Machine Code’ 확인하기

V8은 C++ API를 지원합니다. 때문에, 이 C++ API에서 machine code를 출력해줄 수 있는 API가 있나 확인하려 했습니다.

> 이 C++ API를 통해, Rust에서도 v8을 사용할 수 있으며, 이를 이용해서 javascript를 rust안에서 실행할 수 있습니다.
{: .prompt-info }


### 첫번째 시도

rust에서 v8 crate(package)가 있어서, 이를 통해 complie결과물을 출력하고자 하였습니다.  
[v8 - Rust](https://docs.rs/v8/latest/v8/)

```rust
let platform = v8::new_default_platform(0, false).make_shared();
v8::V8::initialize_platform(platform);
v8::V8::initialize();

let isolate = &mut v8::Isolate::new(Default::default());

let scope = &mut v8::HandleScope::new(isolate);
let context = v8::Context::new(scope, Default::default());
let scope = &mut v8::ContextScope::new(scope, context);

let code = v8::String::new(scope, "'Hello' + ' World!'").unwrap();
println!("javascript code: {}", code.to_rust_string_lossy(scope));

let script = v8::Script::compile(scope, code, None).unwrap();
let result = script.run(scope).unwrap();
let result = result.to_string(scope).unwrap();
println!("result: {}", result.to_rust_string_lossy(scope));
```

여기서 compile결과물에 해당하는 ‘script’변수에 ‘machine code를 확인할 수 있는 method가 있지 않을까?’라고 생각했습니다.  
기대하는 method는 없었습니다.

### 두번째 시도

v8을 debug모드로 build하면, stdout(여기선 console)을 통해, ‘machine code’를 확인할 수 있다고 합니다.  
이를 위해선, 먼저 v8을 직접 build해야 합니다.  
1. v8 build환경 갖추기

   git clone을 통해 v8의 source를 copy해서 시작할 수 있지만,  
   [Building V8 from source · V8](https://v8.dev/docs/build)  
   > Don’t just `git clone` either of these URLs! if you want to build V8 from your checkout, instead follow the instructions below to get everything set up correctly.
   >

   이렇게 ‘정확하게(correctly)’ 설정하기 위해, 별도의 지시사항을 따라야 합니다.

    1. [depot_tools](https://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up)를 세팅해야 합니다.
       > ‘depot_tools’는 chromium작업할때 필요한, git workflow tools를 포함하고 있는 프로젝트입니다.
       {: .prompt-info }

        1. ‘depot_tools’의 소스코드를 clone합니다.

            ```bash
            git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
            ```

        2. 우리가 사용하고 있는 shell이 ‘depot_tools’의 실행파일들을 찾을 수 있도록 환경변수(Environment Variable)를 설정합니다.

            ```bash
            ## bash shell을 사용하는 경우
            echo 'export PATH=$PATH:/path/to/depot_tools' >> ~/.bashrc
            
            ## zsh를 사용하는 경우
            echo 'export PATH=$PATH:/path/to/depot_tools' >> ~/.zshrc
            ```

        3. 이제 shell을 이용해서 어느 경로에 있든 ‘depot_tools’에 있는 기능들을 사용할 수 있습니다.
    2. v8 source code를 fetch합니다.

       > ‘fetch’는 ‘depot_tools’에 포함된 기능입니다.
        {: .prompt-info }

       > 이 과정에서 web browser에서 구글 인증을 요구할 수 있습니다. shell에 뜬 url에 접속하여 인증을 완료하면 됩니다.
       {: .prompt-info }

        ```bash
        mkdir ~/v8 ## v8 folder를 만듭니다.
        cd ~/v8
        fetch v8 ## 'depot_tools'의 fetch기능을 이용해서 v8을 불러옵니다. 이 과정에서 v8 build를 위한 git head가 자동으로 선택됩니다.
        cd v8
        ```


2. v8을 debug모드로 build하기
    1. build를 하기전, v8 source code경로가 있는 곳으로 갑니다.
    2. build를 하기전, 빌드에 사용할 구성을 만들어냅니다.

       ‘tools/dev/v8gen.py’ python script를 사용하여, debug 모드로 bulid하기 위한 구성을 만들어 냅니다.(각종 flag를 설정하기 위함.)

        ```bash
        tools/dev/v8gen.py x64.debug
        ```

       이 결과고 `out.gn/x64.debug` 경로가 생성되었을겁니다.

    3. v8 build flag를 추가해줘야 합니다. 이는 v8이 ‘Machine code’를 stdout에 출력하도록 하기 위함입니다.
       `out.gn/x64.debug/args.gn` 파일에 아래와 같이 flag를 추가합니다.

        ```
        ## file. ./out.gn/x64.debug/args.gn
        
        is_debug = true
        target_cpu = "x64"
        v8_enable_backtrace = true
        v8_enable_slow_dchecks = true
        v8_optimized_debug = false
        
        ## 추가된 flag들
        v8_enable_disassembler = true ## v8이 JIT 컴파일러가 생성한 코드를 disassemble하여 출력할 수 있도록 합니다.
        v8_enable_object_print = true
        v8_enable_verify_heap = true
        ```

    4. a에서 설정한 argument를 기반으로 v8 build하기 (엄청 오래 걸립니다..)

        ```bash
        cd ~/v8 ## v8 source code가 있는 폴더로 이동합니다.
        ninja -C out.gn/x64.debug/ d8
        ```
       >v8의 debugging용 버전임을 나타내기 위해, ‘d8’로 명명해서 사용합니다.
       {: .prompt-info }

        ```bash
        ## execution example
        ✔  13:58 ~/IdeaProjects/v8 [ :32289f80f4d | …4 ] $ ninja -C out.gn/x64.debug/ d8
        ninja: Entering directory `out.gn/x64.debug/'
        [0/1] Regenerating ninja files
        [1893/2449] CXX obj/v8_base_without_compiler/wasm-features.o
        ```

   5. 이제 build한 v8엔진(d8)을 가지고 javascript 파일을 실행합니다.

        ```bash
        out.gn/x64.debug/d8 --print-opt-code --print-code your_script.js
        ```

       `--print-code`: v8이 생선한 모든 코드를 출력합니다.

       `--print-opt-code`: 최적화된 함수에 대한 코드만 출력합니다.

       > 이때, ‘your_script.js’내용이 너무 간단하면, v8은 컴파일하지 않고, 인터프리터를 이용해 실행할 가능성이 높습니다.
       때문에, 최적화 대상이 되도록(JIT를 사용하여 컴파일 하도록) code를 생성해야 합니다.
       {: .prompt-info }

       아래는 example로 사용할 javascript code입니다.

        ```jsx
        function add(a, b) {
          return a + b;
        }
        
        for (let i = 0; i < 100000; i++) {
          add(i, i + 1);
        }
        ```


## 출력 결과

두번째 시도의 최종 shell command를 실행하고 나면, 아래와 같은 출력이 console에 나타납니다.

- Raw source

  실행한 javascript source code를 나타냅니다.

- Optimized code

  JIT 컴파일러로 최적화한 code를 나타냅니다.  
  `compiler = turbofan`: 최적화를 위한 compiler로 [turbofan](https://v8.dev/docs/turbofan)이 사용되었습니다.  
  `Instructions`: 컴파일된 ‘machine code’를 나타냅니다.  

  | 0x178340c00 | 0 | 488d1df9ffffff | REX.W leaq rbx,[rip+0xfffffff9] |
      | --- | --- | --- | --- |
  | 명령어가 저장된 메모리 주소 | byte offset. 이 명령어가 시작 부분에 위치하고 있음을 나타냄. | 우측의 assembly명령어에 해당하는 16진수 값. CPU가 인식하는 명령어 | 머신 코드의 어셈블리 표현. |

```bash
--- Raw source ---
function add(a, b) {
  return a + b;
}

for (let i = 0; i < 100000; i++) {
  add(i, i + 1);
}

--- Optimized code ---
optimization_id = 2
source_position = 0
kind = TURBOFAN
stack_slots = 16
compiler = turbofan
address = 0x134800040375

Instructions (size = 644)
0x178340c00     0  488d1df9ffffff       REX.W leaq rbx,[rip+0xfffffff9]
0x178340c07     7  483bd9               REX.W cmpq rbx,rcx
0x178340c0a     a  740d                 jz 0x178340c19  <+0x19>
0x178340c0c     c  ba84000000           movl rdx,0x84
0x178340c11    11  41ff9568560000       call [r13+0x5668]
0x178340c18    18  cc                   int3l
0x178340c19    19  8b59f4               movl rbx,[rcx-0xc]
0x178340c1c    1c  490b9de0010000       REX.W orq rbx,[r13+0x1e0]
0x178340c23    23  f6431a20             testb [rbx+0x1a],0x20
0x178340c27    27  0f85933407a0         jnz 0x1183b40c0  (CompileLazyDeoptimizedCode)    ;; near builtin entry
0x178340c2d    2d  55                   push rbp
0x178340c2e    2e  4889e5               REX.W movq rbp,rsp
0x178340c31    31  56                   push rsi
0x178340c32    32  57                   push rdi
0x178340c33    33  50                   push rax
0x178340c34    34  ba58000000           movl rdx,0x58
0x178340c39    39  41ff9568560000       call [r13+0x5668]
0x178340c40    40  cc                   int3l
0x178340c41    41  4883ec18             REX.W subq rsp,0x18
0x178340c45    45  488975a0             REX.W movq [rbp-0x60],rsi
0x178340c49    49  493b65a0             REX.W cmpq rsp,[r13-0x60] (external value (StackGuard::address_of_jslimit()))
0x178340c4d    4d  0f8622010000         jna 0x178340d75  <+0x175>
0x178340c53    53  488b4dc0             REX.W movq rcx,[rbp-0x40]
0x178340c57    57  f6c101               testb rcx,0x1
0x178340c5a    5a  0f85ea010000         jnz 0x178340e4a  <+0x24a>
0x178340c60    60  81f9400d0300         cmpl rcx,0x30d40
0x178340c66    66  0f8c1e000000         jl 0x178340c8a  <+0x8a>
0x178340c6c    6c  488b45c8             REX.W movq rax,[rbp-0x38]
0x178340c70    70  488b4de8             REX.W movq rcx,[rbp-0x18]
0x178340c74    74  488be5               REX.W movq rsp,rbp
0x178340c77    77  5d                   pop rbp
0x178340c78    78  4883f901             REX.W cmpq rcx,0x1
0x178340c7c    7c  7f03                 jg 0x178340c81  <+0x81>
0x178340c7e    7e  c20800               ret 0x8
0x178340c81    81  415a                 pop r10
0x178340c83    83  488d24cc             REX.W leaq rsp,[rsp+rcx*8]
0x178340c87    87  4152                 push r10
0x178340c89    89  c3                   retl
0x178340c8a    8a  488bf9               REX.W movq rdi,rcx
0x178340c8d    8d  d1ff                 sarl rdi, 1
0x178340c8f    8f  4c8bc7               REX.W movq r8,rdi
0x178340c92    92  4183c001             addl r8,0x1
0x178340c96    96  0f80b2010000         jo 0x178340e4e  <+0x24e>
0x178340c9c    9c  4103f8               addl rdi,r8
0x178340c9f    9f  0f80ad010000         jo 0x178340e52  <+0x252>
0x178340ca5    a5  41807db100           cmpb [r13-0x4f] (external value (StackGuard::address_of_interrupt_request(StackGuard::InterruptLevel::kNoH
0x178340caa    aa  0f85ee000000         jnz 0x178340d9e  <+0x19e>
0x178340cb0    b0  660f1f840000000000   nop
0x178340cb9    b9  0f1f8000000000       nop
0x178340cc0    c0  4181f8a0860100       cmpl r8,0x186a0
0x178340cc7    c7  0f8d95000000         jge 0x178340d62  <+0x162>
0x178340ccd    cd  498bc8               REX.W movq rcx,r8
0x178340cd0    d0  83c101               addl rcx,0x1
0x178340cd3    d3  0f807d010000         jo 0x178340e56  <+0x256>
0x178340cd9    d9  498bf8               REX.W movq rdi,r8
0x178340cdc    dc  03f9                 addl rdi,rcx
0x178340cde    de  0f8076010000         jo 0x178340e5a  <+0x25a>
0x178340ce4    e4  81f9a0860100         cmpl rcx,0x186a0
0x178340cea    ea  0f8d72000000         jge 0x178340d62  <+0x162>
0x178340cf0    f0  4c8bc1               REX.W movq r8,rcx
0x178340cf3    f3  4183c001             addl r8,0x1
0x178340cf7    f7  0f8061010000         jo 0x178340e5e  <+0x25e>
0x178340cfd    fd  488bf9               REX.W movq rdi,rcx
0x178340d00   100  4103f8               addl rdi,r8
0x178340d03   103  0f8059010000         jo 0x178340e62  <+0x262>
0x178340d09   109  4181f8a0860100       cmpl r8,0x186a0
0x178340d10   110  0f8d4c000000         jge 0x178340d62  <+0x162>
0x178340d16   116  498bc8               REX.W movq rcx,r8
0x178340d19   119  83c101               addl rcx,0x1
0x178340d1c   11c  0f8044010000         jo 0x178340e66  <+0x266>
0x178340d22   122  498bf8               REX.W movq rdi,r8
0x178340d25   125  03f9                 addl rdi,rcx
0x178340d27   127  0f803d010000         jo 0x178340e6a  <+0x26a>
0x178340d2d   12d  81f9a0860100         cmpl rcx,0x186a0
0x178340d33   133  0f8d29000000         jge 0x178340d62  <+0x162>
0x178340d39   139  4c8bc1               REX.W movq r8,rcx
0x178340d3c   13c  4183c001             addl r8,0x1
0x178340d40   140  0f8028010000         jo 0x178340e6e  <+0x26e>
0x178340d46   146  488bf9               REX.W movq rdi,rcx
0x178340d49   149  4103f8               addl rdi,r8
0x178340d4c   14c  0f8020010000         jo 0x178340e72  <+0x272>
0x178340d52   152  41807db100           cmpb [r13-0x4f] (external value (StackGuard::address_of_interrupt_request(StackGuard::InterruptLevel::kNoH
0x178340d57   157  0f8571000000         jnz 0x178340dce  <+0x1ce>
0x178340d5d   15d  e95effffff           jmp 0x178340cc0  <+0xc0>
0x178340d62   162  488bcf               REX.W movq rcx,rdi
0x178340d65   165  03cf                 addl rcx,rdi
0x178340d67   167  0f808e000000         jo 0x178340dfb  <+0x1fb>
0x178340d6d   16d  488bc1               REX.W movq rax,rcx
0x178340d70   170  e9fbfeffff           jmp 0x178340c70  <+0x70>
0x178340d75   175  b9a0000000           movl rcx,0xa0
0x178340d7a   17a  51                   push rcx
0x178340d7b   17b  48bb90dd5d1a01000000 REX.W movq rbx,0x11a5ddd90    ;; external reference (Runtime::StackGuardWithGap)
0x178340d85   185  b801000000           movl rax,0x1
0x178340d8a   18a  48be851a2800ee040000 REX.W movq rsi,0x4ee00281a85    ;; object: 0x04ee00281a85 <NativeContext[300]>
0x178340d94   194  e867ee40a0           call 0x11874fc00  (CEntry_Return1_ArgvOnStack_NoBuiltinExit)    ;; near builtin entry
0x178340d99   199  e9b5feffff           jmp 0x178340c53  <+0x53>
0x178340d9e   19e  48bb50d75d1a01000000 REX.W movq rbx,0x11a5dd750    ;; external reference (Runtime::HandleNoHeapWritesInterrupts)
0x178340da8   1a8  33c0                 xorl rax,rax
0x178340daa   1aa  48be851a2800ee040000 REX.W movq rsi,0x4ee00281a85    ;; object: 0x04ee00281a85 <NativeContext[300]>
0x178340db4   1b4  48897d90             REX.W movq [rbp-0x70],rdi
0x178340db8   1b8  4c894598             REX.W movq [rbp-0x68],r8
0x178340dbc   1bc  e83fee40a0           call 0x11874fc00  (CEntry_Return1_ArgvOnStack_NoBuiltinExit)    ;; near builtin entry
0x178340dc1   1c1  488b7d90             REX.W movq rdi,[rbp-0x70]
0x178340dc5   1c5  4c8b4598             REX.W movq r8,[rbp-0x68]
0x178340dc9   1c9  e9f2feffff           jmp 0x178340cc0  <+0xc0>
0x178340dce   1ce  48897d90             REX.W movq [rbp-0x70],rdi
0x178340dd2   1d2  4c894598             REX.W movq [rbp-0x68],r8
0x178340dd6   1d6  488b1dc3ffffff       REX.W movq rbx,[rip+0xffffffc3]
0x178340ddd   1dd  33c0                 xorl rax,rax
0x178340ddf   1df  48be851a2800ee040000 REX.W movq rsi,0x4ee00281a85    ;; object: 0x04ee00281a85 <NativeContext[300]>
0x178340de9   1e9  e812ee40a0           call 0x11874fc00  (CEntry_Return1_ArgvOnStack_NoBuiltinExit)    ;; near builtin entry
0x178340dee   1ee  488b7d90             REX.W movq rdi,[rbp-0x70]
0x178340df2   1f2  4c8b4598             REX.W movq r8,[rbp-0x68]
0x178340df6   1f6  e9c5feffff           jmp 0x178340cc0  <+0xc0>
0x178340dfb   1fb  c5832ac7             vcvtlsi2sd xmm0,xmm15,rdi
0x178340dff   1ff  498b4d48             REX.W movq rcx,[r13+0x48] (external value (Heap::NewSpaceAllocationTopAddress()))
0x178340e03   203  c5fb1145a0           vmovsd [rbp-0x60],xmm0
0x178340e08   208  488d790c             REX.W leaq rdi,[rcx+0xc]
0x178340e0c   20c  49397d50             REX.W cmpq [r13+0x50] (external value (Heap::NewSpaceAllocationLimitAddress())),rdi
0x178340e10   210  0f8713000000         ja 0x178340e29  <+0x229>
0x178340e16   216  ba0c000000           movl rdx,0xc
0x178340e1b   21b  e8e04707a0           call 0x1183b5600  (AllocateInYoungGeneration)    ;; near builtin entry
0x178340e20   220  488d48ff             REX.W leaq rcx,[rax-0x1]
0x178340e24   224  c5fb1045a0           vmovsd xmm0,[rbp-0x60]
0x178340e29   229  488d790c             REX.W leaq rdi,[rcx+0xc]
0x178340e2d   22d  49897d48             REX.W movq [r13+0x48] (external value (Heap::NewSpaceAllocationTopAddress())),rdi
0x178340e31   231  4883c101             REX.W addq rcx,0x1
0x178340e35   235  c741ff6d050000       movl [rcx-0x1],0x56d
0x178340e3c   23c  c5fb114103           vmovsd [rcx+0x3],xmm0
0x178340e41   241  488bc1               REX.W movq rax,rcx
0x178340e44   244  e927feffff           jmp 0x178340c70  <+0x70>
0x178340e49   249  90                   nop
0x178340e4a   24a  41ff55d0             call [r13-0x30]
0x178340e4e   24e  41ff55d0             call [r13-0x30]
0x178340e52   252  41ff55d0             call [r13-0x30]
0x178340e56   256  41ff55d0             call [r13-0x30]
0x178340e5a   25a  41ff55d0             call [r13-0x30]
0x178340e5e   25e  41ff55d0             call [r13-0x30]
0x178340e62   262  41ff55d0             call [r13-0x30]
0x178340e66   266  41ff55d0             call [r13-0x30]
0x178340e6a   26a  41ff55d0             call [r13-0x30]
0x178340e6e   26e  41ff55d0             call [r13-0x30]
0x178340e72   272  41ff55d0             call [r13-0x30]
0x178340e76   276  41ff55d8             call [r13-0x28]
0x178340e7a   27a  41ff55d8             call [r13-0x28]
0x178340e7e   27e  41ff55d8             call [r13-0x28]
0x178340e82   282  6690                 nop

Inlined functions (count = 1)
 0x04ee00298ee5 <SharedFunctionInfo add>

Deoptimization Input Data (deopt points = 14)
 index  bytecode-offset  node-id    pc
     0               21       31    NA
     1               21       50    NA
     2                2       60    NA
     3               21       88    NA
     4                2       98    NA
     5               21      111    NA
     6                2      122    NA
     7               21      135    NA
     8                2      145    NA
     9               21      158    NA
    10                2      169    NA
    11               47       18   199
    12               47       68   1c1
    13               47      177   1ee

Safepoints (entries = 4, byte size = 32)
0x178340d99    199  slots (sp->fp): 00100011  deopt     11 trampoline:    276
0x178340dc1    1c1  slots (sp->fp): 00100000  deopt     12 trampoline:    27a
0x178340dee    1ee  slots (sp->fp): 00100000  deopt     13 trampoline:    27e
0x178340e20    220  slots (sp->fp): 00000000

RelocInfo (size = 19)
0x178340c29  near builtin entry
0x178340d7d  external reference (Runtime::StackGuardWithGap)  (0x11a5ddd90)
0x178340d8c  full embedded object  (0x04ee00281a85 <NativeContext[300]>)
0x178340d95  near builtin entry
0x178340da0  external reference (Runtime::HandleNoHeapWritesInterrupts)  (0x11a5dd750)
0x178340dac  full embedded object  (0x04ee00281a85 <NativeContext[300]>)
0x178340dbd  near builtin entry
0x178340de1  full embedded object  (0x04ee00281a85 <NativeContext[300]>)
0x178340dea  near builtin entry
0x178340e1c  near builtin entry

--- End code ---
```

## References

**v8 build를 위한 참고문서**
: [Documentation · V8](https://v8.dev/docs)

**Machine code**
: [Machine code](https://en.wikipedia.org/wiki/Machine_code)

**Understanding V8’s Bytecode**
: [Understanding V8’s Bytecode](https://medium.com/dailyjs/understanding-v8s-bytecode-317d46c94775)
