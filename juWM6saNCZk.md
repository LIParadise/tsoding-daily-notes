# Why is C Compiler So Smart?

So if you're writing some code that contains some common patterns, one thing compiler might do is say "ah ha I know this, and if we implicitly do this instead it's better!", and transforms something like this

``` C
void null_some_bytes(char *p)
{
    int32_t n = SOME_MACRO_SIZE;
    while (n-- > 0)
    {
        *p++ = 0;
    }
}
```

So with `-O1`, `gcc` might say:

``` asm
null_some_bytes:
        xorps   xmm0, xmm0
        movups  xmmword ptr [rdi + 80], xmm0
        movups  xmmword ptr [rdi + 64], xmm0
        movups  xmmword ptr [rdi + 48], xmm0
        movups  xmmword ptr [rdi + 32], xmm0
        movups  xmmword ptr [rdi + 16], xmm0
        movups  xmmword ptr [rdi], xmm0
        mov     dword ptr [rdi + 96], 0
        ret
```

Which is totally fine: it's using some arch-specific instruction to speed things up, if `SOME_MACRO_SIZE` were set to be like `100`.
However things get interesting as `SOME_MACRO_SIZE` set to `1000`:

``` C
null_some_bytes:
        push    rax
        xor     esi, esi
        mov     edx, 1000
        call    memset
        pop     rax
        ret
```

It actually replace whole implementation with a **C standard library** function call `memset`!

Which if you're a generic Linux programmer, this is totally fine, but you might find it troublesome when there's no C standard library to link with. The link would just error, e.g. maybe `wasm` or embedded systems.

So one thing you may do is `--no-builtin`, which is telling the compiler not to do these sort of patter recognization, or better `-ffreestanding` which still leaves some pattern matching on, just without replacing the matched patterns with the C standard library calls.

Also, you may want `--no-standard-libraries` to avoid linking with the C standard library.

[TI](https://software-dl.ti.com/codegen/docs/tiarmclang/compiler_tools_user_guide/compiler_manual/using_compiler/compiler_options/language_options.html)
> Avoid linking in the C/C++ standard libraries. This is useful when partially linking an application, or when you want to link against your own standards-compliant libraries.
