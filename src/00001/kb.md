# gfortran, OpenMP and static linking

When linking Fortran executables or linking to `-lgfortran` statically in
conjunction with `-fopenmp` upon termination the application segfaults in the
function `_gfortrani_close_units`.

For reasons that I don't fully understand at the moment it is sufficient to link
pthread in whole-archive mode, i.e. on the linker command line

```
-Wl,--whole-archive -lpthread -Wl,--no-whole-archive
```

See this [GCC Fortran mailing list thread][ml] for details.

[ml]: https://gcc.gnu.org/pipermail/fortran/2021-April/055903.html
