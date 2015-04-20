# WRAPPING `dubset-gracenote` in Ruby with SWIG
-------

--------


In order to wrap `dubset-gracenote` I first got a task to wrap `musicid_file_trackid`, in order to find out which issues I could encounter in wrapping `dubset-gracenote` project (.c file)

## Wrapping `GNSDK/samples/musicid_file_trackid/`
------

### Dividing `main.c` file
First thing that I did was dividing the `main.c` file into `musicid_file_trackid.h` and `musicid_file_trackid.c` files, in that:
1. I seperated the declarations of `"public"` functions into `.h` file
2. deleted the `main()` function and the declarations of the above mentioned functions in `.c` file.
3. 

