# WRAPPING `dubset-gracenote` in Ruby with SWIG
-------

--------


In order to wrap `dubset-gracenote` I first got a task to wrap `musicid_file_trackid`, in order to find out which issues I could encounter in wrapping `dubset-gracenote` project (.c file)

## Wrapping `GNSDK/samples/musicid_file_trackid/`
------

### Dividing `main.c` file

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

As the result of the execution of the `sample` is the same as before diving the file, now we know that we have done this part of the job well, and can proceed to the next step.

### Writing Swig interface `.i` file

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


### Compiling & making the `ruby module`

To compile make the wrapper code using the swig `.i` file (and corresponding .h files), write this in linux shell:

`swig -[target_language] [file_name.i]`

In this particular case, write:

`swig -ruby musicid_file_trackid.i`

By doing this we have made the wrapper `.c` file (`musicid_file_trackid_wrap.c`)

--------

Now, you have to make object (`o`) files from wrapper file `musicid_file_trackid_wrap.c` and original `.c` file (if it existed) `musicid_file_trackid.c`, which you will then use to make a shared object (`.so`) file `musicid_file_trackid.so`.

After making the `.so`, use make install command to make the ruby module. In order to do that, you must fisrt create a ruby `Makefile`.

Commands for doing this:






