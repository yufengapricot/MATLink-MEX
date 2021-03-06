#_MATLink_

_MATLink_ is a [_Mathematica_](http://www.wolfram.com/mathematica/) application that allows the user to communicate data between _Mathematica_ and [MATLAB](http://www.mathworks.com/products/matlab/), as well as execute MATLAB code and seamslesly call MATLAB functions from within _Mathematica_.
It uses [_MathLink_](http://reference.wolfram.com/mathematica/tutorial/MathLinkAndExternalProgramCommunicationOverview.html) to communicate between _Mathematica_ and MATLAB (via the [MATLAB Engine library](http://www.mathworks.com/help/matlab/matlab_external/using-matlab-engine.html)).

##System requirements

MATLink is compatible with Mathematica 8 or later and MATLAB R2011b or later.  

On Linux systems, the C shell `csh` must be installed at `/bin/csh` [for MATLAB Engine applications to work](http://www.mathworks.com/help/matlab/matlab_external/using-matlab-engine.html).  Also on Linux, very old versions of gcc can't be used to compile MATLink.  Please check if your compiler is supported by the version of MATLAB you have.  [For MATLAB R2013a it is gcc 4.4.](http://www.mathworks.com/support/compilers/R2013a/index.html?sec=glnxa64)

##Installation
To be able to use _MATLink_, you will need to install it to a location in _Mathematica_'s `$Path`. You can follow one of the three ways below (replace `$UserBaseDirectory` with whatever is shown when you evaluate it in _Mathematica_):

 - Download the [zip file](https://github.com/rsmenon/MATLink/archive/develop.zip) and extract the contents to `$UserBaseDirectory/Applications/MATLink/`.
 - Clone this repository using `git`:

	``` bash
	git clone https://github.com/rsmenon/MATLink.git $UserBaseDirectory/Applications/MATLink
	```
 - Install from within _Mathematica_ using Leonid Shifrin's [ProjectInstaller](https://github.com/lshifr/ProjectInstaller) package:

	```ruby
	ProjectInstall[URL["https://github.com/rsmenon/MATLink/archive/develop.zip"]]
	```

Some further setup may be necessary to let MATLink find MATLAB:

 - On Windows, this is the standard procedure that is necessary to run MATLAB Engine applications:  1. First, add MATLAB's `bin/win64` (`bin/win32` for 32-bit versions) directory to the system `PATH`.  To do this, follow the instructions [here](http://www.mathworks.com/support/solutions/en/data/1-15ZLK/index.html).  2. Now register the default MATLAB version by running the `regmatlabserver` command from within MATLAB.  On most Windows systems it will be necessary to run MATLAB as administrator for the `regmatlabserver` command to work, but this step needs to be done only once.
 - On OS X, navigate to the `MATLink/Engine/bin/MacOSX64` directory, edit the file `mengine.sh` and set the path to the MATLAB app bundle.
 - On Linux, both MATLAB and Mathematica must be in the system `PATH`.  Then MATLink will be able to automatically compile its binary component (a C++ compiler needs to be installed).

##Quick start guide
###Starting MATLAB
After installing the application, load it using ``Needs["MATLink`"]`` and execute `OpenMATLAB[]`. This will launch an instance of MATLAB with which you can now communicate.

###Executing MATLAB code
Using the `MEvaluate` command, it is possible to execute arbitrary MATLAB code. Simply enter the code as strings and pass it to `MEvaluate`. Note that any output that would be displayed in MATLAB's command window will also be displayed in _Mathematica_, so remember to suppress output with semicolons.

As a simple example, consider the following MATLAB code (modified from the example [here](http://www.mathworks.com/help/matlab/ref/for.html)):

```matlab
k = 4;
hilbert = zeros(k,k);      % Preallocate matrix

for m = 1:k
    for n = 1:k
        hilbert(m,n) = 1/(m+n -1);
    end
end
```

One can evaluate this from within _Mathematica_ as:

```ruby
MEvaluate["
	k = 4;
	hilbert = zeros(k,k);      % Preallocate matrix

	for m = 1:k
		for n = 1:k
			hilbert(m,n) = 1/(m+n -1);
		end
	end
"]
```

<sub>(Note: The indentation is only for clarity; MATLAB ignores stray newlines and whitespace)</sub>

You can verify that the matrix `hilbert` was created by evaluating `MEvaluate["whos hilbert"]`.

###Transferring variables
The functions `MSet` and `MGet` allow the user to transfer variables to and from the MATLAB workspace. The variable name is always passed as a string. Continuing the above example, we can import the matrix `hilbert` to the _Mathematica_ workspace:

```ruby
hilbert = MGet["hilbert"];
```

The result is imported as a native _Mathematica_ array, i.e., a list of lists, which can then be used in further computations.

Similarly, one can pass a variable to MATLAB using the `MSet` function. We shall now pass the imported variable `hilbert` back to MATLAB and store it in a new variable called `gilbert`:

```ruby
MSet["gilbert", hilbert];
```

You can now verify that the imported matrix was indeed passed back correctly by evaluating `MEvaluate["isequal(gilbert,hilbert)"]`

###Saving and executing arbitrary MATLAB code as scripts
Often one wishes to reuse code found online, either on the MathWorks [File Exchange](http://www.mathworks.com/matlabcentral/fileexchange/) or elsewhere and execute it as a script (with side-effects). The function `MScript` makes it easy to do this. In the following example, we define a script (not a function) called `timing.m` that does some computations and displays the timing

```ruby
t = MScript["timing",
	"tic;
	for i=1:100
		eig(randn(100));
	end
	toc"
]
```
<sub>(Note: The indentation is only for clarity; MATLAB ignores stray newlines and whitespace)</sub>

You can now run the above script anytime, several times within the current session, by simply evaluating `MEvaluate[t]` (assuming its value has not been cleared) or `MEvaluate[MScript["timing"]]`.

> **Also see:** "Overwriting session scripts" under the **Advanded usage** section.


###Defining and using MATLAB functions
You can natively call MATLAB functions already on its path using the `MFunction` command, which allows the user to extend the functionality of MATLAB. As an example, we'll define and use the `magic` function from MATLAB, which is not available in _Mathematica_:

```ruby
magic = MFunction["magic"];
magic[4] // MatrixForm
```

To define a custom function for the current session and use it, use `MScript` to save it to a file (remember to use the same filename as the function) and then use `MFunction["function_name"]`, where `function_name` is the name of your function file. As a simple example:

```ruby
MScript["add2", "
	function out = add2(x,y)
  	 	out = x + y;
  	end
 "];
MFunction["add2"][3, 4]
(* Output: 7. *)
```

> **Also see:** "Handling functions with multiple outputs (and no outputs)" under the **Advanded usage** section.

###Closing MATLAB
To completely disconnect from the MATLAB engine, call `DisconnectMATLAB[]`. This will also delete all scripts that were defined for the current _MATLink_ session.

##Advanced usage

###Properly starting and closing MATLAB
There are in fact, four functions that provide different functionality associated with starting and closing MATLAB:

 - `ConnectMATLAB[]` establishes a _MathLink_ connection to the low level C functions and the MATLAB engine, sets various session specific variables, but does not launch MATLAB.
 - `OpenMATLAB[]` launches the MATLAB application in the background.
 - `CloseMATLAB[]` closes the MATLAB application, but keeps the connection to the MATLAB engine open.
 - `DisconnectMATLAB[]` closes the MATLAB engine, terminates the _MathLink_ connection and clears session variables and temporary folders.

For convenience, directly calling `OpenMATLAB[]` automatically calls `ConnectMATLAB[]`, but it is possible to only call `ConnectMATLAB[]` without actually opening MATLAB. One can then call `OpenMATLAB[]` and `CloseMATLAB[]` several times (assuming other external factors such as kernel/front end crashes haven't terminated the _MathLink_ connection) and it is still considered to be the same "session".

The scripts defined during the session are saved in in a session specific folder in the user's `$TemporaryDirectory`. This directory and its contents are removed only when `DisconectMATLAB[]` is called. Hence it is always preferable (and recommended) to call `DisconnectMATLAB[]` when done with using _MATLink_. If the application terminates due to forced kernel quits or crashes, the temporary directory remains, and a new one is created for the next session.

Over prolonged use, these session specific directories can accumulate (since _Mathematica_ crashes are inevitable), if one is not meticulous about regularly clearing their `$TemporaryDirectory`. In such cases, the user can load the package as follows, before connecting to MATLAB:

```ruby
Needs["MATLink`"]
MATLink`Developer`CleanupTemporaryDirectories[]
```

###Overwriting session scripts
By default, it is not possible to overwrite a script defined in the current session using `MScript`. Attempting to do so will produce the following error:

```
MScript::owrt: An MScript by that name already exists. Use "Overwrite" -> True to overwrite.
```

If it is necessary to overwrite the script (to fix typos or change parameters), use the option `"Overwrite" -> True` in `MScript`.

###Handling functions with multiple outputs (and no outputs)
In MATLAB, one can define functions to have completely different behaviour based on the number of _output_ arguments for the same set of input arguments. This is at odds with the behaviour in _Mathematica_ (and in functional programming languages in general), where a function's behaviour is determined solely by its inputs. To bridge this divide, `MFun  ction` offers the functionality to use the multiple output form of MATLAB functions, but the number of outputs must be set explicitly when defining the function.

As an example, consider the `eig` function in MATLAB, which has a single output form that returns only the eigenvalues as a vector, and the two output form which returns both the eigenvalues and the eigenvectors as a matrices. We associate a different symbol in _Mathematica_ for each of those two cases as:

```ruby
eig = MFunction["eig"];
eigsystem = MFunction["eig", "OutputArguments" -> 2];
```

This way, each function in _Mathematica_ still does only one thing for the same set of inputs, but also allows the user to make full use of MATLAB's versatility.

Sometimes, one does not desire an output from a function that produces other side-effects. For example, plotting commands in MATLAB produce a plot, but also return the handle number if an output is requested. To supress the output, one can use the `"Output" -> False` argument. The following example, uses data from _Mathematica_ and creates a MATLAB plot — all from within _Mathematica_:

```ruby
imagesc = MFunction["imagesc", "Output" -> False];
data = Import["ExampleData/ozonemap.hdf", {"Datasets", "TOTAL_OZONE"}][[20 ;;, All]];
imagesc[data];
```
![](http://i.stack.imgur.com/UxUkbm.png)

##Supported MATLAB data types

The following data types can be transferred in both directions:

 - double precision numerical arrays (including multidimensional)
 - double precision sparse matrices
 - logical arrays (including multidimensional)
 - sparse logical matrices
 - strings (i.e. char arrays of dimension `[1 n]`)
 - cells (including multidimensional)
 - structs (only with size `[1 1]`)


The following can only be transferred from MATLAB to Mathematica:

 - numerical arrays with the following types: single, int16, int32
 - structs with any number of elements 


##Known issues and limitations

###Large array support

At the moment, only arrays with less than `2^31-1` elements are supported.  Note that this is true for matrices and multidimensional arrays as well: the _total number_ of matrix elements may not excede `2^31-1` even if the matrix has fewer rows and columns than this.  As an example, the largest supported square matrix can be of size 46341 by 46341.

As a reference point, a double precision array with the maximum number of allowed elements would take up 16 GB of memory, so this limit should be more than sufficient for most applications.

###Inf and NaN

Inf and Nan are not supported at the moment.

###Multiple instances of MATLAB

On OS X, if a MATLAB background process has already been started by _MATLink_, it will not be possible to launch another instance of MATLAB by clicking on its icon.  As a workaround, either start MATLAB before you call `OpenMATLAB[]`or start MATLAB from the terminal as

```bash
open -n /Applications/MATLAB_R2012b.app
```
You can also open it by directly executing the binary from the command line:

```bash
 /Applications/MATLAB_R2012b.app/bin/matlab
```

###`MGet`ting custom classes

Do not use `MGet` on custom classes (things for which `isobject` is true), or data structures that contain custom classes as elements.  On OS X and Unix this will crash the MATLAB process because of a bug in the MATLAB Engine interface.

###Reading HDF5 based `.mat` files

All the limitations of the [MATLAB Engine interface](http://www.mathworks.com/help/matlab/matlab_external/using-matlab-engine.html) apply to MATLink.  The most noticeable of these is that HFD5 based `.mat` files cannot be read.  Quoting the [MATLAB documentation](http://www.mathworks.com/help/matlab/matlab_external/using-matlab-engine.html),

> The MATLAB engine cannot read MAT-files in a format based on HDF5. These are MAT-files saved using the -v7.3 option of the save function or opened using the w7.3 mode argument to the C or Fortran matOpen function.

###Unicode support

`MGet` and `MSet` do support Unicode strings.  However, `MEvaluate` and related functions may not handle them correctly depending on operating system and MATLAB version.

###JIT accelerator

The JIT accelerator does not work for commands submitted using `MEvaluate` on Windows (as of R2013a).  This means that the same MATLAB program may run much faster when submitted using `MScript` than when using `MEvaluate`.   However, the performance difference will only be significant for certain types of code, and is usually non-existent when using simple commands.  Scripts or functions defined in .m files *will* still use the JIT accelerator when called from `MEvaluate`, and vectorized code is not affected much by the JIT.  The only case when you can expect bad performance from `MEvaluate` is if you submit a longer piece of non-vectorized code which heavily relies on loops (`for`, `while`, etc.).

Example of the type of code that will be affected by the lack of acceleration (it relies on explicit loops):

```
x=zeros(1e7,1);
for i=1:length(x)
  x(i)=rand();
end
```

Example of the type of code that will *not* be affected (relying on built-in or custom functions and vectorization):

```
x = rand(1e7,1)
```

Currently only Windows is affected, but future versions of MATLink may lose support for JIT acceleration with `MEvaluate` on all platforms, so please consider using `MScript` for your projects when appropriate.

---

<sub>_Mathematica_ is a registered trademark of Wolfram Research, Inc. and MATLAB is a registered trademark of The MathWorks, Inc.</sub>
