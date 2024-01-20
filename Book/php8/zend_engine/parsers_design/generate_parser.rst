Generate Makefile.frag
=============

Because the generation of lexer/parser call dependant tools of the OS( sed, grep) than is not present in Windows(findstr, powershell) 
The choice of a Makefile.frag generation script was made to limit the number of tools involved in the PHP build toolchain.

This is why you don't need Re2c/Bison if you compile PHP from a package source https://www.php.net/downloads

<php-src>/scripts/dev/genfiles

This is what your Makefile.frag should contain

  $(srcdir)/zend_language_scanner.c: $(srcdir)/zend_language_scanner.l $(srcdir)/zend_language.h
	$(RE2C) $(RE2C_FLAGS) --no-generation-date --case-inverted --bit-vectors --flex-syntax \
	    -c --type-header $(srcdir)/zend_language_scanner_defs.h \
	    -o $(srcdir)/zend_language_scanner.c \
	    $(srcdir)/zend_language_scanner.l

Note than ``$(srcdir)`` if définied in ``configure.ac``; ``$(RE2C)`` and ``$(RE2C_FLAGS)`` also is defined in configure.ac throu the macro ``PHP_SUBSET``

> Normally everything is already done

Generating lexer/parser
=============
Yes, once your <php-src>/Zend/Makefile.frag is up to date you must relaunch <php-src>$ ./buildconf <php-build>$ <php-src>/configure

Now, every time you run the compilation make will automatically generate the parser as needed.

Zend Engine use a .ini parser and a .php parser

  - zend_language_parser.y
  - zend_language_scanner.l

  - zend_ini_parser.y
  - zend_ini_scanner.y

In php, extensions also use parser

  - ext/date/date-iso.y
  - ext/date/date-iso.l
  - ext/date/date.y
  - ext/date/date.l

  - ext/json/parser.y
  - ext/json/lexer.l

There are even extensions that use the Zend Engine language scanner[Aka lexer]

  - ext/tokenizer


Advanced skill
=============
You can go further by creating an m4 macro in <php-src>/build/php.m4
It will unify/simplify the build toolchain.

Customize the parser.y through ``<php-src>/configure --legacy=modern`` without having to restart build
Or backwards-compatibility or what you whant ... --product=[cli|fpm|gui|library] --env=[production|desktop] --target=[release|debug]

	dnl
	dnl PHP_BISON([VERBOSE])
	dnl
	dnl Enable it in desktop environement (disabled by default for small performance reasons)
	dnl Search for zend_language_parser.y and optionally add directive to implement syntaxe error reporting
	dnl Assume Error: type error: in file on line at column
	dnl Provide: extra information to detec error.
	dnl
	AC_DEFUN([PHP_BISON], [
	    # on Linux
	    AC_PATH_PROGS([GREP_COMMAND], [grep], [not-found])
	    # on windows
	    AC_PATH_PROGS([FINDSTR_COMMAND], [findstr], [not-found])
	    
	    # $verbose=$1[no|yes], $pattern=$2, $parser_y=$3
	    AC_MSG_CHECKING([whether to stamp zend_language_parser.stamp])
	    
	    if test "x$GREP_COMMAND" != "xnot-found"; then
	        #result=`m4_esyscmd([grep -q "$2" "$3.y" && echo "echo found" || echo "echo unknow"])`
	        result=$(grep -q $2 $3.y && echo "found" || echo "unknow")
	        
	        if test "x$result" = "xfound"; then
	            if test "x$1" = "xno"; then
	                AC_MSG_RESULT([yes])
	                #set -x
	                #`syscmd([touch "$3.stamp"])`
	                touch "$3.stamp"
	                #set +x
	            else
	                AC_MSG_RESULT([no])
	            fi
	        else
	            if test "x$1" = "xno"; then
	                AC_MSG_RESULT([no])
	            else
	                AC_MSG_RESULT([yes])
	                #set -x
	                #`syscmd([touch "$3.stamp"])`
	                touch "$3.stamp"
	                #set +x
	            fi
	        fi
	    fi
	    
	    if test "x$FINDSTR_COMMAND" != "xnot-found"; then
	        AC_MSG_RESULT(["--enable-verbose is not implemented in windows"])
	        #result=`m4_esyscmd([findstr /L "$2" "$3" >nul 2>&1 && echo "echo found" || echo "echo unknow"])`
	        # sed ? PowerShell ?
	    fi
	])
