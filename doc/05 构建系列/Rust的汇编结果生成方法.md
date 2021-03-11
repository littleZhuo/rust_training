参考文章

[《查看 Golang、Lua、JS、Rust、Python等语言生成的汇编代码》](https://zhuanlan.zhihu.com/p/77158150)
[《rustc手册》](https://learnku.com/docs/rustc-book/2020/command-line-arguments/8860)

具体实例：

```rust
const fn calc_factorial(n: usize) -> usize {
  if n == 0 {
    return 0;
  }

  if n == 1 {
    return 1;
  }

  return n * calc_factorial(n - 1);
}

fn main() {
  println!("{}", calc_factorial(6));
}
```

在命令行执行rustc命令

`rustc -O --emit asm  .\main.rs`

生成main.s的汇编文件，大概内容如下：

```asm
	.text
	.def	 @feat.00;
	.scl	3;
	.type	0;
	.endef
	.globl	@feat.00
.set @feat.00, 0
	.file	"main.7rcbfp3g-cgu.0"
	.def	 _ZN3std10sys_common9backtrace28__rust_begin_short_backtrace17h9b27bcb3ba6bf74fE;
	.scl	3;
	.type	32;
	.endef
	.p2align	4, 0x90
_ZN3std10sys_common9backtrace28__rust_begin_short_backtrace17h9b27bcb3ba6bf74fE:
.seh_proc _ZN3std10sys_common9backtrace28__rust_begin_short_backtrace17h9b27bcb3ba6bf74fE
	subq	$40, %rsp
	.seh_stackalloc 40
	.seh_endprologue
	callq	*%rcx
	leaq	32(%rsp), %rax
	#APP
	#NO_APP
	addq	$40, %rsp
	retq
	.seh_handlerdata
	.text
	.seh_endproc

	.def	 _ZN3std2rt10lang_start17h279602eaf7da0e83E;
	.scl	2;
	.type	32;
	.endef
	.globl	_ZN3std2rt10lang_start17h279602eaf7da0e83E
	.p2align	4, 0x90
_ZN3std2rt10lang_start17h279602eaf7da0e83E:
.seh_proc _ZN3std2rt10lang_start17h279602eaf7da0e83E
	subq	$40, %rsp
	.seh_stackalloc 40
	.seh_endprologue
	movq	%r8, %r9
	movq	%rdx, %r8
	movq	%rcx, 32(%rsp)
	leaq	.L__unnamed_1(%rip), %rdx
	leaq	32(%rsp), %rcx
	callq	_ZN3std2rt19lang_start_internal17he12bde4ee8419433E
	nop
	addq	$40, %rsp
	retq
	.seh_handlerdata
	.text
	.seh_endproc

	.def	 _ZN3std2rt10lang_start28_$u7b$$u7b$closure$u7d$$u7d$17h1248eac854303d55E;
	.scl	3;
	.type	32;
	.endef
	.p2align	4, 0x90
_ZN3std2rt10lang_start28_$u7b$$u7b$closure$u7d$$u7d$17h1248eac854303d55E:
.seh_proc _ZN3std2rt10lang_start28_$u7b$$u7b$closure$u7d$$u7d$17h1248eac854303d55E
	subq	$40, %rsp
	.seh_stackalloc 40
	.seh_endprologue
	movq	(%rcx), %rcx
	callq	_ZN3std10sys_common9backtrace28__rust_begin_short_backtrace17h9b27bcb3ba6bf74fE
	xorl	%eax, %eax
	addq	$40, %rsp
	retq
	.seh_handlerdata
	.text
	.seh_endproc

	.def	 _ZN4core3ops8function6FnOnce40call_once$u7b$$u7b$vtable.shim$u7d$$u7d$17h2aebe20c2941c59bE;
	.scl	3;
	.type	32;
	.endef
	.p2align	4, 0x90
_ZN4core3ops8function6FnOnce40call_once$u7b$$u7b$vtable.shim$u7d$$u7d$17h2aebe20c2941c59bE:
.seh_proc _ZN4core3ops8function6FnOnce40call_once$u7b$$u7b$vtable.shim$u7d$$u7d$17h2aebe20c2941c59bE
	subq	$40, %rsp
	.seh_stackalloc 40
	.seh_endprologue
	movq	(%rcx), %rcx
	callq	_ZN3std10sys_common9backtrace28__rust_begin_short_backtrace17h9b27bcb3ba6bf74fE
	xorl	%eax, %eax
	addq	$40, %rsp
	retq
	.seh_handlerdata
	.text
	.seh_endproc

	.def	 _ZN4core3ptr13drop_in_place17h9dad5e7ee0d878aeE;
	.scl	3;
	.type	32;
	.endef
	.p2align	4, 0x90
_ZN4core3ptr13drop_in_place17h9dad5e7ee0d878aeE:
	retq

	.def	 _ZN4main4main17h2d6c3d678af9e020E;
	.scl	3;
	.type	32;
	.endef
	.p2align	4, 0x90
_ZN4main4main17h2d6c3d678af9e020E:
.seh_proc _ZN4main4main17h2d6c3d678af9e020E
	subq	$104, %rsp
	.seh_stackalloc 104
	.seh_endprologue
	movq	$720, 32(%rsp)
	leaq	32(%rsp), %rax
	movq	%rax, 40(%rsp)
	leaq	_ZN4core3fmt3num3imp54_$LT$impl$u20$core..fmt..Display$u20$for$u20$usize$GT$3fmt17h774508bd3edc8ca4E(%rip), %rax
	movq	%rax, 48(%rsp)
	leaq	.L__unnamed_2(%rip), %rax
	movq	%rax, 56(%rsp)
	movq	$2, 64(%rsp)
	movq	$0, 72(%rsp)
	leaq	40(%rsp), %rax
	movq	%rax, 88(%rsp)
	movq	$1, 96(%rsp)
	leaq	56(%rsp), %rcx
	callq	_ZN3std2io5stdio6_print17hd8fcd78be1f2d333E
	nop
	addq	$104, %rsp
	retq
	.seh_handlerdata
	.text
	.seh_endproc

	.def	 main;
	.scl	2;
	.type	32;
	.endef
	.globl	main
	.p2align	4, 0x90
main:
.seh_proc main
	pushq	%rbp
	.seh_pushreg %rbp
	pushq	%rsi
	.seh_pushreg %rsi
	pushq	%rdi
	.seh_pushreg %rdi
	subq	$48, %rsp
	.seh_stackalloc 48
	leaq	48(%rsp), %rbp
	.seh_setframe %rbp, 48
	.seh_endprologue
	movq	%rdx, %rsi
	movslq	%ecx, %rdi
	callq	__main
	leaq	_ZN4main4main17h2d6c3d678af9e020E(%rip), %rax
	movq	%rax, -8(%rbp)
	leaq	.L__unnamed_1(%rip), %rdx
	leaq	-8(%rbp), %rcx
	movq	%rdi, %r8
	movq	%rsi, %r9
	callq	_ZN3std2rt19lang_start_internal17he12bde4ee8419433E
	nop
	addq	$48, %rsp
	popq	%rdi
	popq	%rsi
	popq	%rbp
	retq
	.seh_handlerdata
	.text
	.seh_endproc

	.section	.rdata,"dr"
	.p2align	3
.L__unnamed_1:
	.quad	_ZN4core3ptr13drop_in_place17h9dad5e7ee0d878aeE
	.quad	8
	.quad	8
	.quad	_ZN3std2rt10lang_start28_$u7b$$u7b$closure$u7d$$u7d$17h1248eac854303d55E
	.quad	_ZN3std2rt10lang_start28_$u7b$$u7b$closure$u7d$$u7d$17h1248eac854303d55E
	.quad	_ZN4core3ops8function6FnOnce40call_once$u7b$$u7b$vtable.shim$u7d$$u7d$17h2aebe20c2941c59bE

.L__unnamed_3:

.L__unnamed_4:
	.byte	10

	.p2align	3
.L__unnamed_2:
	.quad	.L__unnamed_3
	.zero	8
	.quad	.L__unnamed_4
	.asciz	"\001\000\000\000\000\000\000"
```

注意109行，在编译阶段就直接得出了结果 `	movq	$720, 32(%rsp)`
