Generate Makefile.frag
=============

Because the generation of lexer/parser call dependant tools of the OS( sed, grep) than is not present in Windows(findstr, powershell) 
The choice of a Makefile.frag generation script was made to limit the number of tools involved in the PHP build toolchain.

This is why you don't need Re2c/Bison if you compile PHP from a package https://www.php.net/downloads

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
  
  zend_language_parser.y
  zend_language_scanner.l

  zend_ini_parser.y
  zend_ini_scanner.y

  ext/date/date-iso.y
  ext/date/date-iso.l
  ext/date/date.y
  ext/date/date.l

  ext/json/parser.y
  ext/json/lexer.l
