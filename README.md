# Using-Scheme-from-R-via-Guile-and-Rcpp

(define simple-func
  (lambda ()
    (display "Script called, now I can change this") (newline)))

(define quick-test
  (lambda ()
    (display "Adding another function, can modify without recompilation") 
    (newline)
    (adding-without-recompilation)))

(define adding-without-recompilation
  (lambda ()
    (display "Called this, without recompiling the C code") (newline)))

The C wrapper testguile.c

This file is also posted in this gist. We used (as shown below) simpler link instructions which (on Ubuntu) can be ontained via guile-config link and are just -lguile-3.0 -lgc on our Ubuntu 20.04 machine.

/*
 * Compile using: 
 * gcc testguile.c \
 * -I/usr/local/Cellar/guile/1.8.8/include \
 * -D_THREAD_SAFE -L/usr/local/Cellar/guile/1.8.8/lib -lguile -lltdl -lgmp -lm -lltdl -o testguile
 */

#include 
#include 

int main(int argc, char** argv) {
    SCM func, func2;
    scm_init_guile();

    scm_c_primitive_load("script.scm");

    func = scm_variable_ref(scm_c_lookup("simple-func"));
    func2 = scm_variable_ref(scm_c_lookup("quick-test"));

    scm_call_0(func);
    scm_call_0(func2);

    return 0;
}

The Rcpp Caller

This is an edited and updated version of Matías’ initial file.

#include 
#include 
#include 

using namespace Rcpp;

// [[Rcpp::export]]
int test_guile(std::string file) {
    SCM func, func2;
    scm_init_guile();

    scm_c_primitive_load(file.c_str());

    func = scm_variable_ref(scm_c_lookup("simple-func"));
    func2 = scm_variable_ref(scm_c_lookup("quick-test"));

    scm_call_0(func);
    scm_call_0(func2);

    return 0;
}

The System Integration

On our Ubuntu 20.04 box, Guile 3.0 is well supported and is easy installable. Header files are in a public location, as are the library files. The configuration helper guile-config (also guile-config-3.0 to permit continued use of 2.2 and 2.0) provides the required compiler and linker flags which we simply hardwired into src/Makevars. A proper package would need to discover these from a script, typically configure, or Makefile invocations. We leave this for another time; if anybody reading this is interested in taking this further please get in touch.

PKG_CXXFLAGS = -I"/usr/include/guile/3.0"
PKG_LIBS = -lguile-3.0 -lgc

Package

The rest of package was created via one call Rcpp.package.skeleton("RcppGuile") after which we

    simply copied in the C++ file above into src/guile.cpp;
    added the src/Makevars snippet above; and
    copied in the example Scheme scripts as inst/guile/script.scm

and that’s it! Then any of the usual steps with R will build and install a working package.


Guile Demo

The package can be used as follows, retrieving the Scheme script from within the package and clearly running it:

> library(RcppGuile)
> test_guile(system.file("guile", "script.scm", package="RcppGuile"))
Script called, now I can change this
Adding another function, can modify without recompilation
Called this, without recompiling the C code
[1] 0
> 

GNU Guile is made for building extensions. This short note showed how easy it is to connect R with Guile via a simple Rcpp package.
