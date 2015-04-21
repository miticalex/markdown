# Table of Contents
1. [ Dividing main.c file](#1-dividing-mainc-file)
2. [ Writing Swig interface `.i` file](#2-writing-swig-interface-i-file)
3. [ Compiling & making the `ruby module`](#3-compiling--making-the-ruby-module)
4. [ Problems that I encountered in making the `musicid_file_trackid` wrapper and module so far](#4-problems-that-i-encountered-in-making-the-musicid_file_trackid-wrapper-and-module-so-far)

    4.1. [ `Including "gnsdk.h"` header in compiling with `make` (through the process of creating a Makefile)](#41-including-gnsdkh-header-in-compiling-with-make-through-the-process-of-creating-a-makefile)
    
    4.2. [ Conclusion that I don't have to include the whole `gnsdk.h` header file](#42-conclusion-that-i-dont-have-to-include-the-whole-gnsdkh-header-file)
    
    4.3. [ Used, but not defined `static` functions](#43-used-but-not-defined-static-functions)
     
    4.4. [ Wrapping `musicid_file_albumid`](#44-wrapping-musicid_file_albumid)

5. [ What I did so far, how do I plan to continue/finish the job later, and how I suggest the others to finish the job if someone else accepts the responsibility to finish the it.](#5-what-i-did-so-far-how-do-i-plan-to-continuefinish-the-job-later-and-how-i-suggest-the-others-to-finish-the-job-if-someone-else-accepts-the-responsibility-to-finish-the-it)



# WRAPPING `dubset-gracenote` in Ruby with SWIG
-------

--------


In order to wrap `dubset-gracenote` I first got a task to wrap `musicid_file_trackid`, in order to find out which issues I could encounter in wrapping `dubset-gracenote` project (.c file)

## Wrapping `GNSDK/samples/musicid_file_trackid/`
------

### 1. Dividing `main.c` file

First thing that I have to do is to divide the `main.c` file into `musicid_file_trackid.h` and `main.c` files, in that:
1. I have to seperate the declarations of `"public"` functions into `.h` file
2. delete the declarations of the above mentioned functions in `.c` file.

```c
/* musicid_file_trackid.h */
static int
_init_gnsdk(
	const char*          client_id,
	const char*          client_id_tag,
	const char*          client_app_version,
	const char*          license_path,
	int                  use_local,
	gnsdk_user_handle_t* p_user_handle
	);

static void
_shutdown_gnsdk(
	gnsdk_user_handle_t user_handle
	);

static int
_do_sample_trackid(
	gnsdk_user_handle_t user_handle,
	int                 use_local,
	gnsdk_uint32_t      midf_options
	);
```

I `.c` file I put the `_init_gnsdk`, `_shutdown_gnsdk` and `_do_sample_trackid` into comments section.
```
/* main.c */
...
...
...
/*****************************************
 *    Local Function Declarations
 **********************************************/
/* static int           
_init_gnsdk(
	const char*          client_id,
	const char*          client_id_tag,
	const char*          client_app_version,
	const char*          license_path,
	int                  use_local,
	gnsdk_user_handle_t* p_user_handle
	);

static void
_shutdown_gnsdk(
	gnsdk_user_handle_t user_handle
	);

static int
_do_sample_trackid(
	gnsdk_user_handle_t user_handle,
	int                 use_local,
	gnsdk_uint32_t      midf_options
	);
*/
...
...
...
```

Trying and succeeding to compile sample (and afterwards run) from these two separate files (`.h` & `.c`).

After executing `make` using the instructions from the original `makefile` from the gracenote directory, file `sample` is made.. 
```sh
...
...
...
**** ./sample up to date.
```
Now that we have tha `sample` file we can check with the following commands if it gets the tracks info and write them to the shell:

`./sample clientid clientidtag license [local/online]` or with parameters:

`./sample 15534080 95C19687C4B13C876297F1AE9ACEB3FF ../dubset-gracenote/bin/Dubset_GNSDK_Evaluation_License_File.txt online`

Result:

```sh
...
...
...
*File 4 of 6*

	Multiple results.
	Album count: 5
	Match 1 - Album title:		Pretty Polly
	Match 2 - Album title:		Country Blues
	Match 3 - Album title:		Famous Hits by Dock Boggs
	Match 4 - Album title:		Sånger Om Kärlek [Disc 2]
	Match 5 - Album title:		His Folkways Years 1963-1968 [Disc 2]

*File 5 of 6*

	Multiple results.
	Album count: 1
	Match 1 - Album title:		Quest For Fire: Firestarter Vol. 1

*File 6 of 6*

	Multiple results.
	Album count: 5
	Match 1 - Album title:		Quest For Fire: Firestarter Vol. 1
	Match 2 - Album title:		New English File Advanced [Disc 2]
	Match 3 - Album title:		Jazzblues
	Match 4 - Album title:		51st Anniversary The Story of Life [Disc 5]
	Match 5 - Album title:		Quest For Fire
```

As the result of the execution of the `sample` is the same as before diving the file, now we know that we have done this part of the job correctly, and can proceed to the next step.

### 2. Writing Swig interface `.i` file

As I figured out everything I need to do in swig interface `.i` file is to #include `gnsdk.h` library, define GNSDK specific datatypes and declare the three mentioned function, and thereby order SWIG to make the wrapper code for those functions..
It is also needed to define if GNSDK types act as input or output for certain functions.

The `.i` file (pay close attention to the comments):

```i
%module musicid_file_albumid 
%include "typemaps.i" //necessary to include in order to use INPUT, OUTPUT and other typemaps

%inline %{                  // %{ %} braces are used to copy the code inside 'em directly to the wrapper file
typedef void gnsdk_void_t;  // %inline %{ %} does the above mentioned, plus generates the wrapper code
typedef int gnsdk_uint32_t; // for typedefs it is necerssary to do both, because SWIG has to know about
	typedef gnsdk_void_t*        gnsdk_handle_t;    //the types before wrapping it
	#define GNSDK_DECLARE_HANDLE(handle)     struct Phandle##_s { gnsdk_uint32_t magic; }; typedef struct Phandle##_s* handle
//#endif /* GNSDK_HANDLE_T */
	GNSDK_DECLARE_HANDLE( gnsdk_user_handle_t );
%}                          //it is also necessary to put this typedefs before the usages of those types in 
                            // ".i" file
%{
	#include "gnsdk.h"	// decided not to include it because I don't need whole GNSDK library,
                    	// but just a few type definitinitions
	#include "musicid_file_albumid.h"
	typedef unsigned int gnsdk_uint32_t;	
%}


//  #ifndef GNSDK_HANDLE_T      //Wrote before I figured out that i must wrap it with %inline %{
//	#define GNSDK_HANDLE_T      //This part of the code in unnecessary and can be deleted
//	typedef void gnsdk_void_t;
//	typedef int gnsdk_uint32_t;
//	typedef gnsdk_void_t*        gnsdk_handle_t;
//	#define GNSDK_DECLARE_HANDLE(handle)     struct Phandle##_s { gnsdk_uint32_t magic; }; typedef struct Phandle##_s* handle
	//  #endif /* GNSDK_HANDLE_T */
//	GNSDK_DECLARE_HANDLE( gnsdk_user_handle_t );

/*static*/ int      //functions don't need to be static, so I changed that, because it causes
	_init_gnsdk(    //certain problems with wrapping
	const char*          client_id,
	const char*          client_id_tag,
	const char*          client_app_version,
	const char*          license_path,
	int                  use_local,
	gnsdk_user_handle_t* OUTPUT//p_user_handle
	);

/*static*/ void
	_shutdown_gnsdk(
	gnsdk_user_handle_t INPUT//user_handle
	);

/*static*/ int
	_do_sample_trackid(
	gnsdk_user_handle_t INPUT,//user_handle,
	int                 use_local,
	gnsdk_uint32_t      midf_options
	);

```


### 3. Compiling & making the `ruby module`

To compile make the wrapper code using the swig `.i` file (and corresponding .h files), write this in linux shell:

`swig -[target_language] [file_name.i]`

In this particular case, write:

`swig -ruby musicid_file_trackid.i`

By doing this we have made the wrapper `.c` file (`musicid_file_trackid_wrap.c`)

--------

Now, you have to make object (`o`) files from wrapper file `musicid_file_trackid_wrap.c` and original `.c` file (if it existed) `musicid_file_trackid.c`, which you will then use to make a shared object (`.so`) file `musicid_file_trackid.so`.

After making the `.so`, use make install command to make the ruby module. In order to do that, you must fisrt create a ruby `Makefile`.

In order to do this, first create a ruby configuration file in the `musicid_file_trackid` directory, with which you will then create a makefile:

make `conf.rb` file, that should look like this:

```rb
require 'mkmf'
create_makefile('musicid_file_trackid')
```

Execute `conf.rb` file by typing `ruby conf.rb` in shell, to create a makefile... 

Afterwards, you can create an `'.so'` file in two ways:

1. (in this case, this did not work, because I didn't know how to include `gnsdk.h` in creating a `makefile`. That header is needed to compile `main.o`... At least I haven't found a way to make it work. I will explain why later in `Problems that I encountered` section).
 
  > Just type `make` in shell.

    That will first make object `.o` files by compiling `..._wrap.c` and `.c` (if it exists), and it will make the corresponding `.o` files from them... Then it will link a shared `.so` object, in this case `musicid_file_trackid.so`.
 
2. Make `.o` and `.so` files using a c compiler
 
    1. Make object `.o` files by compiling `..._wrap.c` and `.c` source files, with `gcc`, or any other suitable `c compiler`:
    `gcc -c -fPIC source_file.c -I/include_directories`

    For this particular case:
    
        -    `gcc -c -fPIC  musicid_file_trackid_wrap.c -I/home/alexander/.rvm/rubies/ruby-2.2.1/include/ruby-2.2.0 -I/home/alexander/.rvm/rubies/ruby-2.2.1/include/ruby-2.2.0/x86_64-linux` 
        
        (With `-I/` we included the paths where `ruby.h` and `include/config.h` headers are situated. These files are often not in these directories. To find their location on the disc, we can use `find ~ |grep config.h|grep ruby` and `find ~ |grep ruby.h`)
    
        -   `gcc -c -fPIC  main.c -I/../../include`
        
        (With `-I/` we included the path where `gnsdk.h` header is situated) 
        
        -   If you have successufully compiled `.c` files, and created `.o` files, we can procees to the step __2.__ below 

    2. Make `.so` shared object file, using the following commands:
    
    `gcc -shared musicid_file_trackid_wrap.o main.o -o musicid_file_trackid.so`


After making the `.so` file we can create a ruby module by typing `make install`.

Now you can use this module by writing `require "musicid_file_trackid"` and access its functions/methods through module `Musicid_file_trackid`.

`

### 4. Problems that I encountered in making the `musicid_file_trackid` wrapper and module so far
---

`
#### 4.1. `Including "gnsdk.h"` header in compiling with `make` (through the process of creating a Makefile)

To include a path where this header is located, I changes `conf.rb` file in the following way:
```rb
require 'mkmf'
# Use the code from the row below (find_header('','')) if you know the exact location of the file,
# otherwise write three rows that follow (path = File.realdirpath... ...) 
# find_header('gnsdk.h', '/home/alexander/Work/Study/SWIG/GNSDK/include/gnsdk.h') 
path = File.realdirpath('../../../../include', __FILE__)
puts path 
find_header('gnsdk.h', path)
create_makefile('musicid_file_trackid')
```
Even though the file `gnsdk.h` does exist in the `/home/alexander/Work/Study/SWIG/GNSDK/include` directory, I got this answer:
```sh
$ ruby conf.rb
/home/alexander/Work/Study/SWIG/GNSDK/include
checking for gnsdk.h in /home/alexander/Work/Study/SWIG/GNSDK/include... no
```
I really don't know what I did wrong, so I decided to make `.o` and `.so files with `gcc`.

#### 4.2. Conclusion that I don't have to include the whole `gnsdk.h` header file


Meanwhile, I realized that I don't have to include whole `gnsdk.h` header in swig interface `.i` file, because I wrap only a few file definitions (in `main.c` it is of course necessaryto include `gnsdk.h`). It is enough to define the datatypes that are used by the wrapped functions (and corresponding wrapper code for those datatypes):

"musicid_file_trackid.i":
```i
...
%{
    #include "gnsdk.h"    // decided not to include it because I don't need whole GNSDK library,
                        // but just a few type definitinitions
    ...
    ...   
%}
...
...
%inline %{                  // %{ %} braces are used to copy the code inside 'em directly to the wrapper file
typedef void gnsdk_void_t;  // %inline %{ %} does the above mentioned, plus generates the wrapper code
typedef int gnsdk_uint32_t; // for typedefs it is necerssary to do both, because SWIG has to know about
    typedef gnsdk_void_t*        gnsdk_handle_t;    //the types before wrapping it
    #define GNSDK_DECLARE_HANDLE(handle)     struct Phandle##_s { gnsdk_uint32_t magic; }; typedef struct Phandle##_s* handle
    GNSDK_DECLARE_HANDLE( gnsdk_user_handle_t );
%}                          //it is also necessary to put this typedefs before the usages of those types in 
                            // ".i" file
```


#### 4.3. Used, but not defined `static` functions
After these corrections I was still unable to make `.so` file.. I got only the warning that the three wrapped functions were `used but undefined`, so I could make the `.o` file, but when tried to make the `.so` file I got the message that it was impossible to link undefined functions..

That was strange to me, because in order to make the wrapper module, the only necessary thing one has to do is to wrap the declaration of the function.. If the function is not defined problem should occur only when trying to call the function from Ruby. 

To find out what is wrong I decided to write some `analogous example programs` and try to wrap them, and realiyed that I can easily wrap functions that user-defined datatypes, even if they are not declared..

After a few days of battling with those problems I couldn't realize the difference between wrapping my examples and `musicid_file_trackid`, because I really made completely analogous examples. At leaat I thought so...

To gain a better grasp of what and where the problem is I tried  to wrap `musicid_file_albumid` to see if I would encounter the same problems..


#### 4.4. Wrapping `musicid_file_albumid`


As `musicid_file_album_id` and `musicid_file_tracid` used the functions with the same names and arguments, I divided `main.c` using the same pattern, and just coppied the content of `musicid_file_trackid` to `musicid_file_albumid`.
  
The problem remained the same.. So I had to realize what was the difference between the functions from my examples and the functions wrote in `gnsdk.h` samples, and I realized the only difference was that `gnsdk.h` functions were `static`.. As it was not relevant the behavior if the functions were `static` or not, I decided to simple remove the `static` keyword from function declaration and definition, both in `.c` and `.i` files:
    
```
...
...
/*static*/ int      //functions don't need to be static, so I changed that, because it causes
    _init_gnsdk(    //certain problems with wrapping
    const char*          client_id,
    const char*          client_id_tag,
    const char*          client_app_version,
    const char*          license_path,
    int                  use_local,
    gnsdk_user_handle_t* OUTPUT//p_user_handle
    );

/*static*/ void
    _shutdown_gnsdk(
    gnsdk_user_handle_t INPUT//user_handle
    );

/*static*/ int
    _do_sample_trackid(
    gnsdk_user_handle_t INPUT,//user_handle,
    int                 use_local,
    gnsdk_uint32_t      midf_options
    );
...
...
```


After removing the `static` keyword, there was no problem in making the `musicid_file_albumid.so` file with the above mentioned commands. With `make install` I then make a Ruby module which worked. As the problem with `musicid_file_trackid` was the same, after making these changes, `musicid_file_trackid` module was made without a problem.

`

### 5. What I did so far, how do I plan to continue/finish the job later, and how I suggest the others to finish the job if someone else accepts the responsibility to finish the it.

After solving the problems with compiling, includes, static functions, etc. the only thing I managed to do is to     successufully require `musicid_file_trackid` and `musicid_file_albumid`. 
```rb
2.2.1 :001 > require 'musicid_file_albumid'
 => true 
 ```
 ```rb
2.2.1 :001 > require 'musicid_file_trackid'
 => true 
 ```
 I can call the corresponding functions using the `Musicid_file_trackid` module, with commands `Musicid_file_trackid._initgnsdk(arguments)`, `Musicid_file_trackid._do_sample_trackid(arguments)` and `Musicid_file_trackid._shutdown_gnsdk(arguments)`.
 
As I have defined 6th argument `gnsdk user_handle_t*` as output:
 ```i
 /*static*/ int      //functions don't need to be static, so I changed that, because it causes
    _init_gnsdk(    //certain problems with wrapping
    const char*          client_id,
    const char*          client_id_tag,
    const char*          client_app_version,
    const char*          license_path,
    int                  use_local,
    gnsdk_user_handle_t* OUTPUT//p_user_handle
    );
```
 
I should be able to init gnsdk with the following call:
```
err, handle = Musicid_file_trackid._init_gnsdk("15534080","95C19687C4B13C876297F1AE9ACEB3FF","1","../dubset-gracenote/bin/Dubset_GNSDK_Evaluation_License_File.txt",0)
```
That should store the error code in `err` variable, and gnsdk handle pointer that points to `gnsdk_user_handle_t` file/record in `handle` variable. Instead, Ruby returns me the following message:
```
2.2.1 :004 > err, handle = Musicid_file_albumid._init_gnsdk("15534080","95C19687C4B13C876297F1AE9ACEB3FF","1","../dubset-gracenote/bin/Dubset_GNSDK_Evaluation_License_File.txt",0) 
ArgumentError: wrong # of arguments(5 for 6)
	from (irb):4:in `_init_gnsdk'
	from (irb):4
	from /home/alexander/.rvm/rubies/ruby-2.2.1/bin/irb:11:in `<main>'
```
My supposition is that it is necessary to extend `gnsdk_user_handle_t` datatype, and provide it with constructor, with which we can create new `NULL` `gnsdk_user_handle_t*`. For informtation about extending the existing datatype/struct with SWIG, see [5.5.6 Adding member functions to C structures](http://www.swig.org/Doc3.0/SWIGDocumentation.html#SWIG_adding_member_functions)

I think that after creating a `handle` of `gnsdk_user_handle_t*` datatype it would be possible for it to fetch the return value of the function, using the `err, handle = Musicid_file_albumid._init_gnsdk("15534080","95C19687C4B13C876297F1AE9ACEB3FF","1","../dubset-gracenote/bin/Dubset_GNSDK_Evaluation_License_File.txt",0) ` code.

At least it would be possible to initialize handle with the following call: `err = Musicid_file_albumid._init_gnsdk("15534080","95C19687C4B13C876297F1AE9ACEB3FF","1","../dubset-gracenote/bin/Dubset_GNSDK_Evaluation_License_File.txt",0, handle)`, which should store the initialized handle as a side effect.

----


If this doesn't give the wanted result, see [SWIG DOCUMENTATION 3.0](http://www.swig.org/Doc3.0/SWIGDocumentation.html):
- [10 Argument Handling](http://www.swig.org/Doc3.0/SWIGDocumentation.html#Arguments) to customize handling the arguments
- [11 Typemaps](http://www.swig.org/Doc3.0/SWIGDocumentation.html#Arguments) to modify/customize SWIG's behavior.
- [38.5 Input and output parameters](http://www.swig.org/Doc3.0/SWIGDocumentation.html#Ruby_nn32) to customize handling the arguments with SWIG interface file for Ruby.
- [38.7 Typemaps](http://www.swig.org/Doc3.0/SWIGDocumentation.html#Ruby_nn37) to modify/customize SWIG's behavior for Ruby










