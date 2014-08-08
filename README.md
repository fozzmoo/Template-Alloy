NAME
    Template::Alloy - TT2/3, HT, HTE, Tmpl, and Velocity Engine

SYNOPSIS
  Template::Toolkit style usage
        my $t = Template::Alloy->new(
            INCLUDE_PATH => ['/path/to/templates'],
        );

        my $swap = {
            key1 => 'val1',
            key2 => 'val2',
            code => sub { 42 },
            hash => {a => 'b'},
        };

        # print to STDOUT
        $t->process('my/template.tt', $swap)
            || die $t->error;

        # process into a variable
        my $out = '';
        $t->process('my/template.tt', $swap, \$out);

        ### Alloy uses the same syntax and configuration as Template::Toolkit

  HTML::Template::Expr style usage
        my $t = Template::Alloy->new(
            filename => 'my/template.ht',
            path     => ['/path/to/templates'],
        );

        my $swap = {
            key1 => 'val1',
            key2 => 'val2',
            code => sub { 42 },
            hash => {a => 'b'},
        };

        $t->param($swap);

        # print to STDOUT (errors die)
        $t->output(print_to => \*STDOUT);

        # process into a variable
        my $out = $t->output;

        ### Alloy can also use the same syntax and configuration as HTML::Template

  Text::Tmpl style usage
        my $t = Template::Alloy->new;

        my $swap = {
            key1 => 'val1',
            key2 => 'val2',
            code => sub { 42 },
            hash => {a => 'b'},
        };

        $t->set_delimiters('#[', ']#');
        $t->set_strip(0);
        $t->set_values($swap);
        $t->set_dir('/path/to/templates');

        my $out = $t->parse_file('my/template.tmpl');

        my $str = "Foo #[echo $key1]# Bar";
        my $out = $t->parse_string($str);


        ### Alloy uses the same syntax and configuration as Text::Tmpl

  Velocity (VTL) style usage
        my $t = Template::Alloy->new;

        my $swap = {
            key1 => 'val1',
            key2 => 'val2',
            code => sub { 42 },
            hash => {a => 'b'},
        };

        my $out = $t->merge('my/template.vtl', $swap);

        my $str = "#set($foo 1 + 3) ($foo) ($bar) ($!baz)";
        my $out = $t->merge(\$str, $swap);

  Javascript style usage (requires Template::Alloy::JS)
        my $t = Template::Alloy->new;

        my $swap = {
            key1 => 'val1',
            key2 => 'val2',
            code => sub { 42 },
            hash => {a => 'b'},
        };

        my $out = '';
        $t->process_js('my/template.jstem', $swap, \$out);

        my $str = "[% var foo = 1 + 3; write('(' + foo + ') (' + get('key1') + ')'); %]";
        my $out = '';
        $t->process_js(\$str, $swap, \$out);

DESCRIPTION
    "An alloy is a homogeneous mixture of two or more elements"
    (http://en.wikipedia.org/wiki/Alloy).

    Template::Alloy represents the mixing of features and capabilities from
    all of the major mini-language based template systems (support for
    non-mini-language based systems will happen eventually). With
    Template::Alloy you can use your favorite template interface and syntax
    and get features from each of the other major template systems. And
    Template::Alloy is fast - whether your using mod_perl, CGI, or running
    from the commandline. There is even Template::Alloy::JS for getting a
    little more speed when that is necessary.

    Template::Alloy happened by accident (accidentally on purpose). The
    Template::Alloy (Alloy hereafter) was originally a part of the CGI::Ex
    suite that performed simple variable interpolation. It used TT2 style
    variables in TT2 style tags "[% foo.bar %]". That was all the original
    Template::Alloy did. This was fine and dandy for a couple of years. In
    winter of 2005-2006 Alloy was revamped to add a few features. One thing
    led to another and soon Alloy provided for most of the features of TT2
    as well as some from TT3. Template::Alloy now provides a full-featured
    implementation of the Template::Toolkit language.

    After a move to a new company that was using HTML::Template::Expr and
    Text::Tmpl templates, support was investigated and interfaces for
    HTML::Template, HTML::Template::Expr, Text::Tmpl, and Velocity (VTL)
    were added. All of the various engines offer the same features - each
    using a different syntax and interface.

    More recently, the Template::Alloy::JS capabilities were introduced to
    bring Javascript templates to the server side (along with an increase in
    speed if ran in persistent environments).

    Template::Toolkit brought the most to the table. HTML::Template brought
    the LOOP directive. HTML::Template::Expr brought more vmethods and using
    vmethods as top level functions. Text::Tmpl brought the COMMENT
    directive and encouraged speed matching (Text::Tmpl is almost entirely C
    based and is very fast). The Velocity engine brought AUTO_EVAL and
    SHOW_UNDEFINED_INTERP.

    Most of the standard Template::Toolkit documentation covering
    directives, variables, configuration, plugins, filters, syntax, and
    vmethods should apply to Alloy just fine (This pod tries to explain
    everything - but there is too much). See Template::Alloy::TT for a
    listing of the differences between Alloy and TT.

    Most of the standard HTML::Template and HTML::Template::Expr
    documentation covering methods, variables, expressions, and syntax will
    apply to Alloy just fine as well.

    Most of the standard Text::Tmpl documentation applies, as does the
    documentation covering Velocity (VTL).

    So should you use Template::Alloy ? Well, try it out. It may give you no
    visible improvement. Or it could.

BACKEND
    Template::Alloy uses a recursive regex based grammar (early versions
    during the CGI::Ex::Template phase did not). This allows for the
    embedding of opening and closing tags inside other tags (as in [% a =
    "[% 1 + 2 %]" ; a|eval %]). The individual methods such as parse_expr
    and play_expr may be used by external applications to add TT style
    variable parsing to other applications.

    The regex parser returns an AST (abstract syntax tree) of the text,
    directives, variables, and expressions. All of the different template
    syntax options compile to the same AST format. The AST is composed only
    of scalars and arrayrefs and is suitable for sending to JavaScript via
    JSON or sharing with other languages. The parse_tree method is used for
    returning this AST.

    Once at the AST stage, there are two modes of operation. Alloy can
    either operate directly on the AST using the Play role, or it can
    compile the AST to perl code via the Compile role, and then execute the
    code. To use the perl code route, you must set the COMPILE_PERL flag to
    1. If you are running in a cached-in-memory environment such as
    mod_perl, this is the fastest option. If you are running in a
    non-cached-in-memory environment, then using the Play role to run the
    AST is generally faster. The AST method is also more secure as cached
    AST won't ever eval any "perl" (assuming PERL blocks are disabled -
    which is the default).

ROLES
    Template::Alloy has split out its functionality into discrete roles. In
    Template::Toolkit, this functionality is split into separate classes.
    The roles in Template::Alloy simply add on more methods to the main
    class. When Perl 6 arrives, these roles will be translated into true
    Roles.

    The following is a list of roles used by Template::Alloy.

        Template::Alloy::Compile  - Compile-to-perl role
        Template::Alloy::HTE      - HTML::Template::Expr role
        Template::Alloy::Operator - Operator role
        Template::Alloy::Parse    - Parse-to-AST role
        Template::Alloy::Play     - Play-AST role
        Template::Alloy::Stream   - Stream output role
        Template::Alloy::Tmpl     - Text::Tmpl role
        Template::Alloy::TT       - Template::Toolkit role
        Template::Alloy::Velocity - Velocity role
        Template::Alloy::VMethod  - Virtual methods role

        Template::Alloy::JS       - Javascript functionality - available separately

    Template::Alloy automatically loads the roles when they are needed or
    requested - but not sooner (with the exception of the Operator role and
    the VMethod role which are always needed and always loaded). This is
    good for a CGI environment. In mod_perl you may want to preload a role
    to make the most of shared memory. You may do this by passing either the
    role name or a method supplied by that role.

        # import roles necessary for running TT
        use Template::Alloy qw(Parse Play Compile TT);

        # import roles based on methods
        use Template::Alloy qw(parse_tree play_tree compile_tree process);

    Note: importing roles by method names does not import them into that
    namespace - it is autoloading the role and methods into the
    Template::Alloy namespace. To help make this more clear you may use the
    following syntax as well.

        # import roles necessary for running TT
        use Template::Alloy load => qw(Parse Play Compile TT);

        # import roles based on methods
        use Template::Alloy load => qw(process parse_tree play_tree compile_tree);

        # import roles based on methods
        use Template::Alloy
            Parse => 1,
            Play => 1,
            Compile => 1,
            TT => 1;

    Even with all roles loaded Template::Alloy is still relatively small.
    You can load all of the roles (except the JS role) by passing "all" to
    the use statement.

        use Template::Alloy 'all';

        # or
        use Template::Alloy load => 'all';

        # or
        use Template::Alloy all => 1;

    As a final option, Template::Alloy also includes the ability to stand-in
    for other template modules. It is able to do this because it supports
    the majority of the interface of the other template systems. You can do
    this in the following way:

        use Template::Alloy qw(Text::Tmpl HTML::Template);

        # or
        use Template::Alloy load => qw(Text::Tmpl HTML::Template);

        # or
        use Template::Alloy
            'Text::Tmpl'     => 1,
            'HTML::Template' => 1;

    Note that the use statement will die if any of the passed module names
    are already loaded and not subclasses of Template::Alloy. This will
    avoid thinking that you are using Template::Alloy when you really
    aren't. Using the 'all' option won't automatically do this - you must
    mention the "stood-in" modules by name.

    The following modules may be "stood-in" for:

        Template
        Text::Tmpl
        HTML::Template
        HTML::Template::Expr

    This feature is intended to make using Template::Alloy with existing
    code easier. Most cases should work just fine. Almost all syntax will
    just work (except Alloy may make some things work that were previously
    broken). However Template::Alloy doesn't support 100% of the interface
    of any of the template systems. If you are using "features-on-the-edge"
    then you may need to re-write portions of your code that interact with
    the template system.

PUBLIC METHODS
    The following section lists most of the publicly available methods. Some
    less commonly used public methods are listed later in this document.

    "new"
            my $obj = Template::Alloy->new({
                INCLUDE_PATH => ['/my/path/to/content', '/my/path/to/content2'],
            });

        Arguments may be passed as a hash or as a hashref. Returns a
        Template::Alloy object.

        There are currently no errors during Template::Alloy object
        creation. If you are using the HTML::Template interface, this is
        different behavior. The document is not parsed until the output or
        process methods are called.

    "process"
        This is the TT interface for starting processing. Any errors that
        result in the template processing being stopped will be stored and
        available via the ->error method.

            my $t = Template::Alloy->new;
            $t->process($in, $swap, $out)
                || die $t->error;

        Process takes three arguments.

        The $in argument can be any one of:

            String containing the filename of the template to be processed.
            The filename should be relative to INCLUDE_PATH.  (See
            INCLUDE_PATH, ABSOLUTE, and RELATIVE configuration items).  In
            memory caching and file side caching are available for this type.

            A reference to a scalar containing the contents of the template to be processed.

            A coderef that will be called to return the contents of the template.

            An open filehandle that will return the contents of the template when read.

        The $swap argument should be hashref containing key value pairs that
        will be available to variables swapped into the template. Values can
        be hashrefs, hashrefs of hashrefs and so on, arrayrefs, arrayrefs of
        arrayrefs and so on, coderefs, objects, and simple scalar values
        such as numbers and strings. See the section on variables.

        The $out argument can be any one of:

            undef - meaning to print the completed template to STDOUT.

            String containing a filename.  The completed template will be placed in the file.

            A reference to a string.  The contents will be appended to the scalar reference.

            A coderef.  The coderef will be called with the contents as a single argument.

            An object that can run the method "print".  The contents will be passed as
            a single argument to print.

            An arrayref.  The contents will be pushed onto the array.

            An open filehandle.  The contents will be printed to the open handle.

        Additionally - the $out argument can be configured using the OUTPUT
        configuration item.

        The process method defaults to using the "cet" syntax which will
        parse TT3 and most TT2 documents. To parse HT or HTE documents, you
        must pass the SYNTAX configuration item to the "new" method. All
        calls to process would then default to HTE syntax.

            my $obj = Template::Alloy->new(SYNTAX => 'hte');

    "process_simple"
        Similar to the process method but with the following restrictions:

        The $in parameter is limited to a filename or a reference a string
        containing the contents.

        The $out parameter may only be a reference to a scalar string that
        output will be appended to.

        Additionally, the following configuration variables will be ignored:
        VARIABLES, PRE_DEFINE, BLOCKS, PRE_PROCESS, PROCESS, POST_PROCESS,
        AUTO_RESET, OUTPUT.

    "error"
        Should something go wrong during a "process" command, the error that
        occurred can be retrieved via the error method.

            $obj->process('somefile.html', {a => 'b'}, \$string_ref)
                || die $obj->error;

    "output"
        HTML::Template way to process a template. The output method requires
        that a filename, filehandle, scalarref, or arrayref argument was
        passed to the new method. All of the HT calling conventions for new
        are supported. The key difference is that Alloy will not actually
        process the template until the output method is called.

            my $obj = Template::Alloy->new(filename => 'myfile.html');
            $obj->param(\%swap);
            print $obj->output;

        See the HTML::Template documentation for more information.

        The output method defaults to using the "hte" syntax which will
        parse HTE and HT documents. To parse TT3 or TT2 documents, you must
        pass the SYNTAX configuration item to the "new" method. All calls to
        process would then default to TT3 syntax.

            my $obj = Template::Alloy->new(SYNTAX => 'tt3');

        Any errors that occur during the output method will die with the
        error as the die value.

    "param"
        HTML::Template way to get or set variable values that will be used
        by the output method.

            my $val = $obj->param('key'); # get one value

            $obj->param(key => $val);     # set one value

            $obj->param(key => $val, key2 => $val2);   # set multiple

            $obj->param({key => $val, key2 => $val2}); # set multiple

        See the HTML::Template documentation for more information.

        Note: Alloy does not support the die_on_bad_params configuration.
        This is because Alloy does not resolve variable names until the
        output method is called.

    "define_vmethod"
        This method is available for defining extra Virtual methods or
        filters. This method is similar to Template::Stash::define_vmethod.

            Template::Alloy->define_vmethod(
                'text',
                reverse => sub { my $item = shift; return scalar reverse $item },
            );

    "register_function"
        This is the HTML::Template way of defining text vmethods. It is the
        same as calling define_vmethod with "text" as the first argument.

            Template::Alloy->register_function(
                'reverse', sub { my $item = shift; return scalar reverse $item },
            );

    "define_directive"
        This method can be used for adding new directives or overridding
        existing ones.

           Template::Alloy->define_directive(
               MYDIR => {
                   parse_sub => sub {}, # parse additional items in the tag
                   play_sub  => sub {
                       my ($self, $ref, $node, $out_ref) = @_;
                       $$out_ref .= "I always say the same thing!";
                       return;
                   },
                   is_block  => 1,  # is this block like
                   is_postop => 0,  # not a post operative directive
                   no_interp => 1,  # no interpolation in this block
                   continues => undef, # it doesn't "continue" any other directives
               },
           );

        Now with a template like:

           my $str = "([% MYDIR %]This is something[% END %])";
           Template::Alloy->new->process(\$str);

        You will get:

           (I always say the same thing!)

        We'll add more details in later revisions of this document.

    "define_syntax"
        This method can be used for adding another syntax to or overriding
        existing ones in the list of choices available in Alloy. The syntax
        can be chosen by the SYNTAX configuration item.

            Template::Alloy->define_syntax(
                my_uber_syntax => sub {
                    my $self = shift;
                    local $self->{'V2PIPE'}      = 0;
                    local $self->{'V2EQUALS'}    = 0;
                    local $self->{'PRE_CHOMP'}   = 0;
                    local $self->{'POST_CHOMP'}  = 0;
                    local $self->{'NO_INCLUDES'} = 0;
                    return $self->parse_tree_tt3(@_);
                },
            );

        The subroutine that is used must return an opcode tree (AST) that
        can be played by the execute_tree method.

    "define_operator"
        This method allows for adding new operators or overriding existing
        ones.

            Template::Alloy->define_operator({
                type       => 'right', # can be one of prefix, postfix, right, left, none, ternary, assign
                precedence => 84,      # relative precedence for resolving multiple operators without parens
                symbols    => ['foo', 'FOO'], # any mix of chars can be used for the operators
                play_sub   => sub {
                    my ($one, $two) = @_;
                    return "You've been foo'ed ($one, $two)";
                },
            });

        You can then use it in a template as in the following:

           my $str = "[% 'ralph' foo 1 + 2 * 3 %]";
           Template::Alloy->new->process(\$str);

        You will get:

           You've been foo'ed (ralph, 7)

        Future revisions of this document will include more samples. This is
        an experimental feature and the API will probably change.

    "dump_parse_tree"
        This method allows for returning a Data::Dumper dump of a parsed
        template. It is mainly used for testing.

    "dump_parse_expr"
        This method allows for returning a Data::Dumper dump of a parsed
        variable. It is mainly used for testing.

    "import"
        All of the arguments that can be passed to "use" that are listed
        above in the section dealing with ROLES, can be used with the import
        method.

            # import by role
            Template::Alloy->import(qw(Compile Play Parse TT));

            # import by method
            Template::Alloy->import(qw(compile_tree play_tree parse_tree process));

            # import by "stand-in" class
            Template::Alloy->import('Text::Tmpl', 'HTML::Template::Expr');

        As mentioned in the ROLE section - arguments passed to import are
        not imported into current namespace. Roles and methods are only
        imported into the Template::Alloy namespace.

VARIABLES
    This section discusses how to use variables and expressions in the TT
    mini-language.

    A variable is the most simple construct to insert into the TT mini
    language. A variable name will look for the matching value inside
    Template::Alloys internal stash of variables which is essentially a hash
    reference. This stash is initially populated by either passing a hashref
    as the second argument to the process method, or by setting the
    "VARIABLES" or "PRE_DEFINE" configuration variables.

    If you are using either the HT or the HTE syntax, the VAR, IF, UNLESS,
    LOOP, and INCLUDE directives will accept a NAME attribute which may only
    be a single level (non-chained) HTML::Template variable name, or they
    may accept an EXPR attribute which may be any valid TT3 variable or
    expression.

    The following are some sample ways to access variables.

        ### some sample variables
        my %vars = (
            one       => '1.0',
            foo       => 'bar',
            vname     => 'one',
            some_code => sub { "You passed me (".join(', ', @_).")" },
            some_data => {
                a     => 'A',
                bar   => 3234,
                c     => [3, 1, 4, 1, 5, 9],
                vname => 'one',
            },
            my_list   => [20 .. 50],
            cet       => Template::Alloy->new,
        );

        ### pass the variables into the Alloy process
        $cet->process($template_name, \%vars)
             || die $cet->error;

        ### pass the variables during object creation (will be available to every process call)
        my $cet = Template::Alloy->new(VARIABLES => \%vars);

  GETTING VARIABLES
    Once you have variables defined, they can be used directly in the
    template by using their name in the stash. Or by using the GET
    directive.

        [% foo %]
        [% one %]
        [% GET foo %]

    Would print when processed:

        bar
        1.0
        bar

    To access members of a hashref or an arrayref, you can chain together
    the names using a ".".

        [% some_data.a %]
        [% my_list.0] [% my_list.1 %] [% my_list.-1 %]
        [% some_data.c.2 %]

    Would print:

        A
        20 21 50
        4

    If the value of a variable is a code reference, it will be called. You
    can add a set of parenthesis and arguments to pass arguments. Arguments
    are variables and can be as complex as necessary.

        [% some_code %]
        [% some_code() %]
        [% some_code(foo) %]
        [% some_code(one, 2, 3) %]

    Would print:

        You passed me ().
        You passed me ().
        You passed me (bar).
        You passed me (1.0, 2, 3).

    If the value of a variable is an object, methods can be called using the
    "." operator.

        [% cet %]

        [% cet.dump_parse_expr('1 + 2').replace('\s+', ' ') %]

    Would print something like:

        Template::Alloy=HASH(0x814dc28)

        $VAR1 = [ [ undef, '+', '1', '2' ], 0 ];

    Each type of data (string, array and hash) have virtual methods
    associated with them. Virtual methods allow for access to functions that
    are commonly used on those types of data. For the full list of built in
    virtual methods, please see the section titled VIRTUAL METHODS

        [% foo.length %]
        [% my_list.size %]
        [% some_data.c.join(" | ") %]

    Would print:

        3
        31
        3 | 1 | 4 | 5 | 9

    It is also possible to "interpolate" variable names using a "$". This
    allows for storing the name of a variable inside another variable. If a
    variable name is a little more complex it can be embedded inside of "${"
    and "}".

        [% $vname %]
        [% ${vname} %]
        [% ${some_data.vname} %]
        [% some_data.$foo %]
        [% some_data.${foo} %]

    Would print:

        1.0
        1.0
        1.0
        3234
        3234

    In Alloy it is also possible to embed any expression (non-directive) in
    "${" and "}" and it is possible to use non-integers for array access.
    (This is not available in TT2)

        [% ['a'..'z'].${ 2.3 } %]
        [% {ab => 'AB'}.${ 'a' ~ 'b' } %]
        [% color = qw/Red Blue/; FOR [1..4] ; color.${ loop.index % color.size } ; END %]

    Would print:

        c
        AB
        RedBlueRedBlue

  SETTING VARIABLES.
    To define variables during processing, you can use the = operator. In
    most cases this is the same as using the SET directive.

        [% a = 234 %][% a %]
        [% SET b = "Hello" %][% b %]

    Would print:

        234
        Hello

    It is also possible to create arrayrefs and hashrefs.

        [% a = [1, 2, 3] %]
        [% b = {key1 => 'val1', 'key2' => 'val2'} %]

        [% a.1 %]
        [% b.key1 %] [% b.key2 %]

    Would print:

        2
        val1 val2

    It is possible to set multiple values in the same SET directive.

        [% SET a = 'A'
               b = 'B'
               c = 'C' %]
        [% a %]    [% b %]    [% c %]

    Would print:

        A    B    C

    It is also possible to unset variables, or to set members of nested data
    structures.

        [% a = 1 %]
        [% SET a %]

        [% b.0.c = 37 %]

        ([% a %])
        [% b.0.c %]

    Would print

        ()
        37

LITERALS AND CONSTRUCTORS
    The following are the types of literals (numbers and strings) and
    constructors (hash and array constructs) allowed in Alloy. They can be
    used as arguments to functions, in place of variables in directives, and
    in place of variables in expressions. In Alloy it is also possible to
    call virtual methods on literal values.

    Integers and Numbers.
            [% 23423   %]        Prints an integer.
            [% 3.14159 %]        Prints a number.
            [% pi = 3.14159 %]   Sets the value of the variable.
            [% 3.13159.length %] Prints 7 (the string length of the number)

        Scientific notation is supported.

            [% 314159e-5 + 0 %]      Prints 3.14159.

            [% .0000001.fmt('%.1e') %]  Prints 1.0e-07

        Hexadecimal input is also supported.

            [% 0xff + 0 %]    Prints 255

            [% 48875.fmt('%x') %]  Prints beeb

    Single quoted strings.
        Returns the string. No variable interpolation happens.

            [% 'foobar' %]          Prints "foobar".
            [% '$foo\n' %]          Prints "$foo\\n".  # the \\n is a literal "\" and an "n"
            [% 'That\'s nice' %]    Prints "That's nice".
            [% str = 'A string' %]  Sets the value of str.
            [% 'A string'.split %]  Splits the string on ' ' and returns the list.

        Note: virtual methods can only be used on literal strings in Alloy,
        not in TT.

        You may also embed the current tags in strings (Alloy only).

            [% '[% 1 + 2 %]' | eval %]  Prints "3"

    Double quoted strings.
        Returns the string. Variable interpolation happens.

            [% "foobar" %]                   Prints "foobar".
            [% "$foo"   %]                   Prints "bar" (assuming the value of foo is bar).
            [% "${foo}" %]                   Prints "bar" (assuming the value of foo is bar).
            [% "foobar\n" %]                 Prints "foobar\n".  # the \n is a newline.
            [% str = "Hello" %]              Sets the value of str.
            [% "foo".replace('foo','bar') %] Prints "bar".

        Note: virtual methods can only be used on literal strings in Alloy,
        not in TT.

        You may also embed the current tags in strings (Alloy only).

            [% "[% 1 + 2 %]" | eval %]  Prints "3"

    Array Constructs.
            [% [1, 2, 3] %]               Prints something like ARRAY(0x8309e90).
            [% array1 = [1 .. 3] %]       Sets the value of array1.
            [% array2 = [foo, 'a', []] %] Sets the value of array2.
            [% [4, 5, 6].size %]          Prints 3.
            [% [7, 8, 9].reverse.0 %]     Prints 9.

        Note: virtual methods can only be used on array contructs in Alloy,
        not in TT.

    Quoted Array Constructs.
            [% qw/1 2 3/ %]                Prints something like ARRAY(0x8309e90).
            [% array1 = qw{Foo Bar Baz} %] Sets the value of array1.
            [% qw[4 5 6].size %]           Prints 3.
            [% qw(Red Blue).reverse.0 %]   Prints Blue.

        Note: this works in Alloy and is planned for TT3.

    Hash Constructs.
            [% {foo => 'bar'} %]                 Prints something like HASH(0x8305880)
            [% hash = {foo => 'bar', c => {}} %] Sets the value of hash.
            [% {a => 'A', b => 'B'}.size %]      Prints 2.
            [% {'a' => 'A', 'b' => 'B'}.size %]  Prints 2.
            [% name = "Tom" %]
            [% {Tom => 'You are Tom',
                Kay => 'You are Kay'}.$name %]   Prints You are Tom

        Note: virtual methods can only be used on hash contructs in Alloy,
        not in TT.

    Regex Constructs.
            [% /foo/ %]                              Prints (?-xism:foo)
            [% a = /(foo)/i %][% "FOO".match(a).0 %] Prints FOO

        Note: this works in Alloy and is planned for TT3.

VIRTUAL METHODS
    Virtual methods (vmethods) are a TT feature that allow for operating on
    the swapped template variables.

    This document shows some samples of using vmethods. For a full listing
    of available virtual methods, see Template::Alloy::VMethod.

EXPRESSIONS
    Expressions are one or more variables or literals joined together with
    operators. An expression can be used anywhere a variable can be used
    with the exception of the variable name in the SET directive, and the
    filename of PROCESS, INCLUDE, WRAPPER, and INSERT.

    For a full listing of operators, see Template::Alloy::Operator.

    The following section shows some samples of expressions. For a full list
    of available operators, please see the section titled OPERATORS.

        [% 1 + 2 %]           Prints 3
        [% 1 + 2 * 3 %]       Prints 7
        [% (1 + 2) * 3 %]     Prints 9

        [% x = 2 %]                      # assignments don't return anything
        [% (x = 2) %]         Prints 2   # unless they are in parens
        [% y = 3 %]
        [% x * (y - 1) %]     Prints 4

DIRECTIVES
    This section contains the alphabetical list of DIRECTIVES available in
    Alloy. DIRECTIVES are the "functions" and control structures that work
    in the various mini-languages. For further discussion and examples
    beyond what is listed below, please refer to the TT directives
    documentation or to the appropriate documentation for the particular
    directive.

    The examples given in this section are done using the Template::Toolkit
    syntax, but can be done in any of the various syntax options. See
    Template::Alloy::TT, Template::Alloy::HTE, Template::Alloy::Tmpl, and
    Template::Alloy::Velocity.

        [% IF 1 %]One[% END %]
        [% FOREACH a = [1 .. 3] %]
            a = [% a %]
        [% END %]

        [% SET a = 1 %][% SET a = 2 %][% GET a %]

    In TT multiple directives can be inside the same set of '[%' and '%]'
    tags as long as they are separated by space or semi-colons (;) (The
    Alloy version of Tmpl allows multiple also - but none of the other
    syntax options do). Any block directive that can also be used as a
    post-operative directive (such as IF, WHILE, FOREACH, UNLESS, FILTER,
    and WRAPPER) must be separated from preceding directives with a
    semi-colon if it is being used as a block directive. It is more safe to
    always use a semi-colon. Note: separating by space is only available in
    Alloy but is a planned TT3 feature.

        [% SET a = 1 ; SET a = 2 ; GET a %]
        [% SET a = 1
           SET a = 2
           GET a
         %]

        [% GET 1
             IF 0   # is a post-operative
           GET 2 %] # prints 2

        [% GET 1;
           IF 0     # it is block based
             GET 2
           END
         %]         # prints 1

    The following is the list of directives.

    "BLOCK"
        Saves a block of text under a name for later use in PROCESS,
        INCLUDE, and WRAPPER directives. Blocks may be placed anywhere
        within the template being processed including after where they are
        used.

            [% BLOCK foo %]Some text[% END %]
            [% PROCESS foo %]

            Would print

            Some text

            [% INCLUDE foo %]
            [% BLOCK foo %]Some text[% END %]

            Would print

            Some text

        Anonymous BLOCKS can be used for capturing.

            [% a = BLOCK %]Some text[% END %][% a %]

            Would print

            Some text

        Anonymous BLOCKS can be used with macros.

    "BREAK"
        Alias for LAST. Used for exiting FOREACH and WHILE loops.

    "CALL"
        Calls the variable (and any underlying coderefs) as in the GET
        method, but always returns an empty string.

    "CASE"
        Used with the SWITCH directive. See the "SWITCH" directive.

    "CATCH"
        Used with the TRY directive. See the "TRY" directive.

    "CLEAR"
        Clears any of the content currently generated in the innermost block
        or template. This can be useful when used in conjunction with the
        TRY statement to clear generated content if an error occurs later.

    "COMMENT"
        Will comment out any text found between open and close tags. Note,
        that the intermediate items are still parsed and END tags must align
        - but the parsed content will be discarded.

            [% COMMENT %]
               This text won't be shown.
               [% IF 1 %]And this won't either.[% END %]
            [% END %]

    "CONFIG"
        Allow for changing the value of some compile time and runtime
        configuration options.

            [% CONFIG
                ANYCASE   => 1
                PRE_CHOMP => '-'
            %]

        The following compile time configuration options may be set:

            ANYCASE
            AUTO_EVAL
            AUTO_FILTER
            CACHE_STR_REFS
            ENCODING
            INTERPOLATE
            POST_CHOMP
            PRE_CHOMP
            SEMICOLONS
            SHOW_UNDEFINED_INTERP
            SYNTAX
            V1DOLLAR
            V2EQUALS
            V2PIPE

        The following runtime configuration options may be set:

            ADD_LOCAL_PATH
            CALL_CONTEXT
            DUMP
            VMETHOD_FUNCTIONS
            STRICT (can only be enabled, cannot be disabled)

        If non-named parameters as passed, they will show the current
        configuration:

           [% CONFIG ANYCASE, PRE_CHOMP %]

           CONFIG ANYCASE = undef
           CONFIG PRE_CHOMP = undef

    "DEBUG"
        Used to reset the DEBUG_FORMAT configuration variable, or to turn
        DEBUG statements on or off. This only has effect if the DEBUG_DIRS
        or DEBUG_ALL flags were passed to the DEBUG configuration variable.

            [% DEBUG format '($file) (line $line) ($text)' %]
            [% DEBUG on %]
            [% DEBUG off %]

    "DEFAULT"
        Similar to SET, but only sets the value if a previous value was not
        defined or was zero length.

            [% DEFAULT foo = 'bar' %][% foo %] => 'bar'

            [% foo = 'baz' %][% DEFAULT foo = 'bar' %][% foo %] => 'baz'

    "DUMP"
        DUMP inserts a Data::Dumper printout of the variable or expression.
        If no argument is passed it will dump the entire contents of the
        current variable stash (with private keys removed).

        The output also includes the current file and line number that the
        DUMP directive was called from.

        See the DUMP configuration item for ways to customize and control
        the output available to the DUMP directive.

            [% DUMP %] # dumps everything

            [% DUMP 1 + 2 %]

    "ELSE"
        Used with the IF directive. See the "IF" directive.

    "ELSIF"
        Used with the IF directive. See the "IF" directive.

    "END"
        Used to end a block directive.

    "EVAL"
        Same as the EVALUATE directive.

    "EVALUATE"
        Introduced by the Velocity templating language. Parses and processes
        the contents of the passed item. This is similar to the eval filter,
        but Velocity needs a directive. Named arguments may be used for
        re-configuring the parser. Any of the items that can be passed to
        the CONFIG directive may be passed here.

            [% EVALUATE "[% 1 + 3 %]" %]

            [% foo = "bar" %]
            [% EVALUATE "<TMPL_VAR foo>" SYNTAX => 'ht' %]

    "FILTER"
        Used to apply different treatments to blocks of text. It may operate
        as a BLOCK directive or as a post operative directive. Alloy
        supports all of the filters in Template::Filters. The lines between
        scalar virtual methods and filters is blurred (or non-existent) in
        Alloy. Anything that is a scalar virtual method may be used as a
        FILTER.

        TODO - enumerate the at least 7 ways to pass and use filters.

    '|' Alias for the FILTER directive. Note that | is similar to the '.' in
        Template::Alloy. Therefore a pipe cannot be used directly after a
        variable name in some situations (the pipe will act only on that
        variable). This is the behavior employed by TT3. To get the TT2
        behavior for a PIPE, use the V2PIPE configuration item.

    "FINAL"
        Used with the TRY directive. See the "TRY" directive.

    "FOR"
        Alias for FOREACH

    "FOREACH"
        Allows for iterating over the contents of any arrayref. If the
        variable is not an arrayref, it is automatically promoted to one.

            [% FOREACH i IN [1 .. 3] %]
                The variable i = [% i %]
            [%~ END %]

            [% a = [1 .. 3] %]
            [% FOREACH j IN a %]
                The variable j = [% j %]
            [%~ END %]

        Would print:

                The variable i = 1
                The variable i = 2
                The variable i = 3

                The variable j = 1
                The variable j = 2
                The variable j = 3

        You can also use the "=" instead of "IN" or "in".

            [% FOREACH i = [1 .. 3] %]
                The variable i = [% i %]
            [%~ END %]

            Same as before.

        Setting into a variable is optional.

            [% a = [1 .. 3] %]
            [% FOREACH a %] Hi [% END %]

        Would print:

             hi  hi  hi

        If the item being iterated is a hashref and the FOREACH does not set
        into a variable, then values of the hashref are copied into the
        variable stash.

            [% FOREACH [{a => 1}, {a => 2}] %]
                Key a = [% a %]
            [%~ END %]

        Would print:

                Key a = 1
                Key a = 2

        The FOREACH process uses the Template::Alloy::Iterator class to
        handle iterations (It is compatible with Template::Iterator). During
        the FOREACH loop an object blessed into the iterator class is stored
        in the variable "loop".

        The loop variable provides the following information during a
        FOREACH:

            index  - the current index
            max    - the max index of the list
            size   - the number of items in the list
            count  - index + 1
            number - index + 1
            first  - true if on the first item
            last   - true if on the last item
            next   - return the next item in the list
            prev   - return the previous item in the list
            odd    - return 1 if the current count is odd, 0 otherwise
            even   - return 1 if the current count is even, 0 otherwise
            parity - return "odd" if the current count is odd, "even" otherwise

        The following:

            [% FOREACH [1 .. 3] %] [% loop.count %]/[% loop.size %] [% END %]

        Would print:

             1/3  2/3  3/3

        The iterator is also available using a plugin. This allows for
        access to multiple "loop" variables in a nested FOREACH directive.

            [%~ USE outer_loop = Iterator(["a", "b"]) %]
            [%~ FOREACH i = outer_loop %]
                [%~ FOREACH j = ["X", "Y"] %]
                   [% outer_loop.count %]-[% loop.count %] = ([% i %] and [% j %])
                [%~ END %]
            [%~ END %]

        Would print:

                   1-1 = (a and X)
                   1-2 = (a and Y)
                   2-1 = (b and X)
                   2-2 = (b and Y)

        FOREACH may also be used as a post operative directive.

            [% "$i" FOREACH i = [1 .. 5] %] => 12345

    "GET"
        Return the value of a variable or expression.

            [% GET a %]

        The GET keyword may be omitted.

            [% a %]

            [% 7 + 2 - 3 %] => 6

        See the section on VARIABLES.

    "IF (IF / ELSIF / ELSE)"
        Allows for conditional testing. Expects an expression as its only
        argument. If the expression is true, the contents of its block are
        processed. If false, the processor looks for an ELSIF block. If an
        ELSIF's expression is true then it is processed. Finally it looks
        for an ELSE block which is processed if none of the IF or ELSIF's
        expressions were true.

            [% IF a == b %]A equaled B[% END %]

            [% IF a == b -%]
                A equaled B
            [%- ELSIF a == c -%]
                A equaled C
            [%- ELSE -%]
                Couldn't determine that A equaled anything.
            [%- END %]

        IF may also be used as a post operative directive.

            [% 'A equaled B' IF a == b %]

        Note: If you are using HTML::Template style documents, the TMPL_IF
        tag parses using the limited HTML::Template parsing rules. However,
        you may use EXPR="" to embed a TT3 style expression.

    "INCLUDE"
        Parse the contents of a file or block and insert them. Variables
        defined or modifications made to existing variables are discarded
        after a template is included.

            [% INCLUDE path/to/template.html %]

            [% INCLUDE "path/to/template.html" %]

            [% file = "path/to/template.html" %]
            [% INCLUDE $file %]

            [% BLOCK foo %]This is foo[% END %]
            [% INCLUDE foo %]

        Arguments may also be passed to the template:

            [% INCLUDE "path/to/template.html" a = "An arg" b = "Another arg" %]

        Filenames must be relative to INCLUDE_PATH unless the ABSOLUTE or
        RELATIVE configuration items are set.

        Multiple filenames can be passed by separating them with a plus, a
        space, or commas (TT2 doesn't support the comma). Any supplied
        arguments will be used on all templates.

            [% INCLUDE "path/to/template.html",
                       "path/to/template2.html" a = "An arg" b = "Another arg" %]

        On Perl 5.6 on some platforms there may be some issues with the
        variable localization. There is no problem on 5.8 and greater.

    "INSERT"
        Insert the contents of a file without template parsing.

        Filenames must be relative to INCLUDE_PATH unless the ABSOLUTE or
        RELATIVE configuration items are set.

        Multiple filenames can be passed by separating them with a plus, a
        space, or commas (TT2 doesn't support the comma).

            [% INSERT "path/to/template.html",
                      "path/to/template2.html" %]

    "JS"
        Only available if the COMPILE_JS configuration item is true (default
        is false). This requires the Template::Alloy::JS module to be
        installed.

        Allow eval'ing the block of text as javascript. The block will be
        parsed and then eval'ed.

            [% a = "BimBam" %]
            [%~ JS %]
                write('The variable a was "' + get('a') + '"');
                set('b', "FooBar");
            [% END %]
            [% b %]

        Would print:

            The variable a was "BimBam"
            FooBar

    "LAST"
        Used to exit out of a WHILE or FOREACH loop.

    "LOOP"
        This directive operates similar to the HTML::Template loop
        directive. The LOOP directive expects a single variable name. This
        variable name should point to an arrayref of hashrefs. The keys of
        each hashref will be added to the variable stash when it is
        iterated.

            [% var a = [{b => 1}, {b => 2}, {b => 3}] %]

            [% LOOP a %] ([% b %]) [% END %]

        Would print:

             (1)  (2)  (3)

        If Alloy is in HT mode and GLOBAL_VARS is false, the contents of the
        hashref will be the only items available during the loop iteration.

        If LOOP_CONTEXT_VARS is true, and $QR_PRIVATE is false (default when
        called through the output method), then the variables __first__,
        __last__, __inner__, __odd__, and __counter__ will be set. See the
        HTML::Template loop_context_vars configuration item for more
        information.

    "MACRO"
        Takes a directive and turns it into a variable that can take
        arguments.

            [% MACRO foo(i, j) BLOCK %]You passed me [% i %] and [% j %].[% END %]

            [%~ foo("a", "b") %]
            [% foo(1, 2) %]

        Would print:

            You passed me a and b.
            You passed me 1 and 2.

        Another example:

            [% MACRO bar(max) FOREACH i = [1 .. max] %]([% i %])[% END %]

            [%~ bar(4) %]

        Would print:

            (1)(2)(3)(4)

        Starting with version 1.012 of Template::Alloy there is also a macro
        operator.

            [% foo = ->(i,j){ "You passed me $i and $j" } %]

            [% bar = ->(max){ FOREACH i = [1 .. max]; i ; END } %]

        See the Template::Alloy::Operator documentation for more examples.

    "META"
        Used to define variables that will be available via either the
        template or component namespace.

        Once defined, they cannot be overwritten.

            [% template.foobar %]
            [%~ META foobar = 'baz' %]
            [%~ META foobar = 'bing' %]

        Would print:

            baz

    "NEXT"
        Used to go to the next iteration of a WHILE or FOREACH loop.

    "PERL"
        Only available if the EVAL_PERL configuration item is true (default
        is false).

        Allow eval'ing the block of text as perl. The block will be parsed
        and then eval'ed.

            [% a = "BimBam" %]
            [%~ PERL %]
                my $a = "[% a %]";
                print "The variable \$a was \"$a\"";
                $stash->set('b', "FooBar");
            [% END %]
            [% b %]

        Would print:

            The variable $a was "BimBam"
            FooBar

        During execution, anything printed to STDOUT will be inserted into
        the template. Also, the $stash and $context variables are set and
        are references to objects that mimic the interface provided by
        Template::Context and Template::Stash. These are provided for
        compatibility only. $self contains the current Template::Alloy
        object.

    "PROCESS"
        Parse the contents of a file or block and insert them. Unlike
        INCLUDE, no variable localization happens so variables defined or
        modifications made to existing variables remain after the template
        is processed.

            [% PROCESS path/to/template.html %]

            [% PROCESS "path/to/template.html" %]

            [% file = "path/to/template.html" %]
            [% PROCESS $file %]

            [% BLOCK foo %]This is foo[% END %]
            [% PROCESS foo %]

        Arguments may also be passed to the template:

            [% PROCESS "path/to/template.html" a = "An arg" b = "Another arg" %]

        Filenames must be relative to INCLUDE_PATH unless the ABSOLUTE or
        RELATIVE configuration items are set.

        Multiple filenames can be passed by separating them with a plus, a
        space, or commas (TT2 doesn't support the comma). Any supplied
        arguments will be used on all templates.

            [% PROCESS "path/to/template.html",
                       "path/to/template2.html" a = "An arg" b = "Another arg" %]

    "RAWPERL"
        Only available if the EVAL_PERL configuration item is true (default
        is false). Similar to the PERL directive, but you will need to
        append to the $output variable rather than just calling PRINT.

    "RETURN"
        Used to exit the innermost block or template and continue processing
        in the surrounding block or template.

        There are two changes from TT2 behavior. First, In Alloy, a RETURN
        during a MACRO call will only exit the MACRO. Second, the RETURN
        directive takes an optional variable name or expression, if passed,
        the MACRO will return this value instead of the normal text from the
        MACRO. The process_simple method will also return this value.

        You can also use the item, list, and hash return vmethods.

            [% RETURN %]       # just exits
            [% RETURN "foo" %] # return value is foo
            [% "foo".return %] # same thing

    "SET"
        Used to set variables.

           [% SET a = 1 %][% a %]             => "1"
           [% a = 1 %][% a %]                 => "1"
           [% b = 1 %][% SET a = b %][% a %]  => "1"
           [% a = 1 %][% SET a %][% a %]      => ""
           [% SET a = [1, 2, 3] %][% a.1 %]   => "2"
           [% SET a = {b => 'c'} %][% a.b %]  => "c"

    "STOP"
        Used to exit the entire process method (out of all blocks and
        templates). No content will be processed beyond this point.

    "SWITCH"
        Allow for SWITCH and CASE functionality.

           [% a = "hi" %]
           [% b = "bar" %]
           [% SWITCH a %]
               [% CASE "foo"           %]a was foo
               [% CASE b               %]a was bar
               [% CASE ["hi", "hello"] %]You said hi or hello
               [% CASE DEFAULT         %]I don't know what you said
           [% END %]

        Would print:

           You said hi or hello

    "TAGS"
        Change the type of enclosing braces used to delineate template tags.
        This remains in effect until the end of the enclosing block or
        template or until the next TAGS directive. Either a named set of
        tags must be supplied, or two tags themselves must be supplied.

            [% TAGS html %]

            [% TAGS <!-- --> %]

        The named tags are (duplicated from TT):

            asp       => ['<%',     '%>'    ], # ASP
            default   => ['\[%',    '%\]'   ], # default
            html      => ['<!--',   '-->'   ], # HTML comments
            mason     => ['<%',     '>'     ], # HTML::Mason
            metatext  => ['%%',     '%%'    ], # Text::MetaText
            php       => ['<\?',    '\?>'   ], # PHP
            star      => ['\[\*',   '\*\]'  ], # TT alternate
            template  => ['\[%',    '%\]'   ], # Normal Template Toolkit
            template1 => ['[\[%]%', '%[%\]]'], # allow TT1 style
            tt2       => ['\[%',    '%\]'   ], # TT2

        If custom tags are supplied, by default they are escaped using
        quotemeta. You may also pass explicitly quoted strings, or regular
        expressions as arguments as well (if your regex begins with a ', ",
        or / you must quote it.

            [% TAGS [<] [>] %]          matches "[<] tag [>]"

            [% TAGS '[<]' '[>]' %]      matches "[<] tag [>]"

            [% TAGS "[<]" "[>]" %]      matches "[<] tag [>]"

            [% TAGS /[<]/ /[>]/ %]      matches "< tag >"

            [% TAGS ** ** %]            matches "** tag **"

            [% TAGS /**/ /**/ %]        Throws an exception.

        You should be sure that the start tag does not include grouping
        parens or INTERPOLATE will not function properly.

    "THROW"
        Allows for throwing an exception. If the exception is not caught via
        the TRY DIRECTIVE, the template will abort processing of the
        directive.

            [% THROW mytypes.sometime 'Something happened' arg1 => val1 %]

        See the TRY directive for examples of usage.

    "TRY"
        The TRY block directive will catch exceptions that are thrown while
        processing its block (It cannot catch parse errors unless they are
        in included files or evaltt'ed strings. The TRY block will then look
        for a CATCH block that will be processed. While it is being
        processed, the "error" variable will be set with the thrown
        exception as the value. After the TRY block - the FINAL block will
        be ran whether or not an error was thrown (unless a CATCH block
        throws an error).

        Note: Parse errors cannot be caught unless they are in an eval
        FILTER, or are in a separate template being INCLUDEd or PROCESSed.

            [% TRY %]
            Nothing bad happened.
            [% CATCH %]
            Caught the error.
            [% FINAL %]
            This section runs no matter what happens.
            [% END %]

        Would print:

            Nothing bad happened.
            This section runs no matter what happens.

        Another example:

            [% TRY %]
            [% THROW "Something happened" %]
            [% CATCH %]
              Error:               [% error %]
              Error.type:          [% error.type %]
              Error.info:          [% error.info %]
            [% FINAL %]
              This section runs no matter what happens.
            [% END %]

        Would print:

              Error:               undef error - Something happened
              Error.type:          undef
              Error.info:          Something happened
              This section runs no matter what happens.

        You can give the error a type and more information including named
        arguments. This information replaces the "info" property of the
        exception.

            [% TRY %]
            [% THROW foo.bar "Something happened" "grrrr" foo => 'bar' %]
            [% CATCH %]
              Error:               [% error %]
              Error.type:          [% error.type %]
              Error.info:          [% error.info %]
              Error.info.0:        [% error.info.0 %]
              Error.info.1:        [% error.info.1 %]
              Error.info.args.0:   [% error.info.args.0 %]
              Error.info.foo:      [% error.info.foo %]
            [% END %]

        Would print something like:

              Error:               foo.bar error - HASH(0x82a395c)
              Error.type:          foo.bar
              Error.info:          HASH(0x82a395c)
              Error.info.0:        Something happened
              Error.info.1:        grrrr
              Error.info.args.0:   Something happened
              Error.info.foo:      bar

        You can also give the CATCH block a type to catch. And you can nest
        TRY blocks. If types are specified, Alloy will try and find the
        closest matching type. Also, an error object can be re-thrown using
        $error as the argument to THROW.

            [% TRY %]
              [% TRY %]
                [% THROW foo.bar "Something happened" %]
              [% CATCH bar %]
                Caught bar.
              [% CATCH DEFAULT %]
                Caught default - but re-threw.
                [% THROW $error %]
              [% END %]
            [% CATCH foo %]
              Caught foo.
            [% CATCH foo.bar %]
              Caught foo.bar.
            [% CATCH %]
              Caught anything else.
            [% END %]

        Would print:

                Caught default - but re-threw.

              Caught foo.bar.

    "UNLESS"
        Same as IF but condition is negated.

            [% UNLESS 0 %]hi[% END %]  => hi

        Can also be a post operative directive.

    "USE"
        Allows for loading a Template::Toolkit style plugin.

            [% USE iter = Iterator(['foo', 'bar']) %]
            [%~ iter.get_first %]
            [% iter.size %]

        Would print:

            foo
            2

        Note that it is possible to send arguments to the new object
        constructor. It is also possible to omit the variable name being
        assigned. In that case the name of the plugin becomes the variable.

            [% USE Iterator(['foo', 'bar', 'baz']) %]
            [%~ Iterator.get_first %]
            [% Iterator.size %]

        Would print:

            foo
            3

        Plugins that are loaded are looked up for in the namespace listed in
        the PLUGIN_BASE directive which defaults to Template::Plugin. So in
        the previous example, if Template::Toolkit was installed, the iter
        object would loaded by the class Template::Plugin::Iterator. In
        Alloy, an effective way to disable plugins is to set the PLUGIN_BASE
        to a non-existent base such as "_" (In TT it will still fall back to
        look in Template::Plugin).

        Note: The iterator plugin will fall back and use
        Template::Alloy::Iterator if Template::Toolkit is not installed. No
        other plugins come installed with Template::Alloy.

        The names of the Plugin being loaded from PLUGIN_BASE are case
        insensitive. However, using case insensitive names is bad as it
        requires scanning the @INC directories for any module matching the
        PLUGIN_BASE and caching the result (OK - not that bad).

        If the plugin is not found and the LOAD_PERL directive is set, then
        Alloy will try and load a module by that name (note: this type of
        lookup is case sensitive and will not scan the @INC dirs for a
        matching file).

            # The LOAD_PERL directive should be set to 1
            [% USE ta = Template::Alloy %]
            [%~ ta.dump_parse_expr('2 * 3') %]

        Would print:

            [[undef, '*', 2, 3], 0];

        See the PLUGIN_BASE, and PLUGINS configuration items.

        See the documentation for Template::Manual::Plugins.

    "VIEW"
        Implement a TT style view. For more information, please see the
        Template::View documentation. This DIRECTIVE will correctly parse
        the arguments and then pass them along to a newly created
        Template::View object. It will fail if Template::View can not be
        found.

    "WHILE"
        Will process a block of code while a condition is true.

            [% WHILE i < 3 %]
                [%~ i = i + 1 %]
                i = [% i %]
            [%~ END %]

        Would print:

                i = 1
                i = 2
                i = 3

        You could also do:

            [% i = 4 %]
            [% WHILE (i = i - 1) %]
                i = [% i %]
            [%~ END %]

        Would print:

                i = 3
                i = 2
                i = 1

        Note that (f = f - 1) is a valid expression that returns the value
        of the assignment. The parenthesis are not optional.

        WHILE has a built in limit of 1000 iterations. This is controlled by
        the global variable $WHILE_MAX in Template::Alloy.

        WHILE may also be used as a post operative directive.

            [% "$i" WHILE (i = i + 1) < 7 %] => 123456

    "WRAPPER"
        Block directive. Processes contents of its block and then passes
        them in the [% content %] variable to the block or filename listed
        in the WRAPPER tag.

            [% WRAPPER foo b = 23 %]
            My content to be processed ([% b %]).[% a = 2 %]
            [% END %]

            [% BLOCK foo %]
            A header ([% a %]).
            [% content %]
            A footer ([% a %]).
            [% END %]

        This would print.

            A header (2).
            My content to be processed (23).
            A footer (2).

        The WRAPPER directive may also be used as a post operative
        directive.

            [% BLOCK baz %]([% content %])[% END -%]
            [% "foobar" WRAPPER baz %]

        Would print

            (foobar)');

        Multiple filenames can be passed by separating them with a plus, a
        space, or commas (TT2 doesn't support the comma). Any supplied
        arguments will be used on all templates. Wrappers are processed in
        reverse order, so that the first wrapper listed will surround each
        subsequent wrapper listed. Variables from inner wrappers are
        available to the next wrapper that surrounds it.

            [% WRAPPER "path/to/outer.html",
                       "path/to/inner.html" a = "An arg" b = "Another arg" %]

DIRECTIVES (HTML::Template Style)
    HTML::Template templates use directives that look similar to the
    following:

        <TMPL_VAR NAME="foo">

        <TMPL_IF NAME="bar">
          BAR
        </TMPL_IF>

    The normal set of HTML::Template directives are TMPL_VAR, TMPL_IF,
    TMPL_ELSE, TMPL_UNLESS, TMPL_INCLUDE, and TMPL_LOOP. These tags should
    have either a NAME attribute, an EXPR attribute, or a bare variable name
    that is used to specify the value to be operated. If a NAME is
    specified, it may only be a single level value (as opposed to a TT
    chained variable). In the case of the TMPL_INCLUDE directive, the NAME
    is the file to be included.

    In Alloy, the EXPR attribute can be used with any of these types to
    specify TT compatible variable or expression that will be used for the
    value.

        <TMPL_VAR NAME="foo">          Prints the value contained in foo
        <TMPL_VAR foo>                 Prints the value contained in foo
        <TMPL_VAR EXPR="foo">          Prints the value contained in foo

        <TMPL_VAR NAME="foo.bar.baz">  Prints the value contained in {'foo.bar.baz'}
        <TMPL_VAR EXPR="foo.bar.baz">  Prints the value contained in {foo}->{bar}->{baz}

        <TMPL_IF foo>                  Prints FOO if foo is true
          FOO
        </TMPL_IF

        <TMPL_UNLESS foo>              Prints FOO unless foo is true
          FOO
        </TMPL_UNLESS

        <TMPL_INCLUDE NAME="foo.ht">   Includes the template in "foo.ht"

        <TMPL_LOOP foo>                Iterates on the arrayref foo
          <TMPL_VAR name>
        </TMPL_LOOP>

    Template::Alloy makes all of the other TT3 directives available in
    addition to the normal set of HTML::Template directives. For example,
    the following is valid in Alloy.

        <TMPL_MACRO bar(n) BLOCK>You said <TMPL_VAR n></TMPL_MACRO>
        <TMPL_GET bar("hello")>

    The TMPL_VAR tag may also include an optional ESCAPE attribute. This
    specifies how the value of the tag should be escaped prior to
    substituting into the template.

        Escape value |   Type of escape
        ---------------------------------
        HTML, 1      |   HTML encoding
        URL          |   URL encoding
        JS           |   basic javascript encoding (\n, \r, and \")
        NONE, 0      |   No encoding (default).

    The TMPL_VAR tag may also include an optional DEFAULT attribute that
    contains a string that will be used if the variable returns false.

        <TMPL_VAR foo DEFAULT="Foo was false">

CHOMPING
    Chomping refers to the handling of whitespace immediately before and
    immediately after template tags. By default, nothing happens to this
    whitespace. Modifiers can be placed just inside the opening and just
    before the closing tags to control this behavior.

    Additionally, the PRE_CHOMP and POST_CHOMP configuration variables can
    be set and will globally control all chomping behavior for tags that do
    not have their own chomp modifier. PRE_CHOMP and POST_CHOMP can be set
    to any of the following values:

        none:      0   +   Template::Constants::CHOMP_NONE
        one:       1   -   Template::Constants::CHOMP_ONE
        collapse:  2   =   Template::Constants::CHOMP_COLLAPSE
        greedy:    3   ~   Template::Constants::CHOMP_GREEDY

    CHOMP_NONE
        Don't do any chomping. The "+" sign is used to indicate CHOMP_NONE.

            Hello.

            [%+ "Hi." +%]

            Howdy.

        Would print:

            Hello.

            Hi.

            Howdy.

    CHOMP_ONE (formerly known as CHOMP_ALL)
        Delete any whitespace up to the adjacent newline. The "-" is used to
        indicate CHOMP_ONE.

            Hello.

            [%- "Hi." -%]

            Howdy.

        Would print:

            Hello.
            Hi.
            Howdy.

    CHOMP_COLLAPSE
        Collapse adjacent whitespace to a single space. The "=" is used to
        indicate CHOMP_COLLAPSE.

            Hello.

            [%= "Hi." =%]

            Howdy.

        Would print:

            Hello. Hi. Howdy.

    CHOMP_GREEDY
        Remove all adjacent whitespace. The "~" is used to indicate
        CHOMP_GREEDY.

            Hello.

            [%~ "Hi." ~%]

            Howdy.

        Would print:

            Hello.Hi.Howdy.

CONFIGURATION
    The following configuration variables are supported (in alphabetical
    order). Note: for further discussion you can refer to the TT config
    documentation.

    Items may be passed in upper or lower case. If lower case names are
    passed they will be resolved to uppercase during the "new" method.

    All of the variables in this section can be passed to the "new"
    constructor.

        my $obj = Template::Alloy->new(
            VARIABLES  => \%hash_of_variables,
            AUTO_RESET => 0,
            TRIM       => 1,
            POST_CHOMP => "=",
            PRE_CHOMP  => "-",
        );

    ABSOLUTE
        Boolean. Default false. Are absolute paths allowed for included
        files.

    ADD_LOCAL_PATH
        If true, allows calls include_filename to temporarily add the
        directory of the current template being processed to the
        INCLUDE_PATHS arrayref. This allows templates to refer to files in
        the local template directory without specifying the local directory
        as part of the filename. Default is 0. If set to a negative value,
        the current directory will be added to the end of the current
        INCLUDE_PATHS.

        This property may also be set in the template using the CONFIG
        directive.

            [% CONFIG ADD_LOCAL_PATH => 1 %]

    ANYCASE
        Allow directive matching to be case insensitive.

            [% get 23 %] prints 23 with ANYCASE => 1

    AUTO_RESET
        Boolean. Default 1. Clear blocks that were set during the process
        method.

    AUTO_EVAL
        Boolean. Default 0 (default 1 in Velocity syntax). If set to true,
        double quoted strings will automatically be passed to the eval
        filter. This configuration option may also be passed to the CONFIG
        directive.

    AUTO_FILTER
        Can be the name of any filter. Default undef. Any variable returned
        by a GET directive (including implicit GET) will be passed to the
        named filter. This configuration option may also be passed to the
        CONFIG directive.

            # with AUTO_FILTER => 'html'

            [% f = "&"; GET f %] prints &amp;
            [% f = "&"; f %]     prints &amp; (implicit GET)

        If a variable already has another filter applied the AUTO_FILTER is
        not applied. The "none" scalar virtual method has been added to
        allow for using variables without reapplying filters.

            # with AUTO_FILTER => 'html'

            [% f = "&";  f | none %] prints &
            [% f = "&"; g = f; g %]  prints &amp;
            [% f = "&"; g = f; g | none %]  prints & (because g = f is a SET directive)
            [% f = "&"; g = GET f; g | none %]  prints &amp; (because the actual GET directive was called)

    BLOCKS
        Only available via when using the process interface.

        A hashref of blocks that can be used by the process method.

            BLOCKS => {
                block_1 => sub { ... }, # coderef that returns a block
                block_2 => 'A String',  # simple string
            },

        Note that a Template::Document cannot be supplied as a value (TT
        supports this). However, it is possible to supply a value that is
        equal to the hashref returned by the load_template method.

    CACHE_SIZE
        Number of compiled templates to keep in memory. Default undef.
        Undefined means to allow all templates to cache. A value of 0 will
        force no caching. The cache mechanism will clear templates that have
        not been used recently.

    CACHE_STR_REFS
        Default 1. If set, any string refs will have an MD5 sum taken that
        will then be used for caching the document - both in memory and on
        the file system (if configured). This will give a significant speed
        boost. Note that this affects strings passed to the EVALUATE
        directive or eval filters as well. It may be set using the CONFIG
        directive.

    CALL_CONTEXT (Not in TT)
        Can be one of 'item', 'list', or 'smart'. The default type is
        'smart'. The CALL_CONTEXT configuration specifies in what Perl
        context coderefs and methods used in the processed templates will be
        called. TT historically has avoided the distinction of item (scalar)
        vs list context. To avoid worrying about this, TT introduced 'smart'
        context. The "@()" and "$()" context specifiers make it easier to
        use CALL_CONTEXT in some situations.

        The following table shows the relationship between the various
        contexts:

               return values      smart context   list context    item context
               -------------      -------------   ------------    ------------
            A   'foo'              'foo'           ['foo']         'foo'
            B   undef              undef           [undef]         undef
            C   (no return value)  undef           []              undef
            D   (7)                7               [7]             7
            E   (7,8,9)            [7,8,9]         [7,8,9]         9
            F   @a = (7)           7               [7]             1
            G   @a = (7,8,9)       [7,8,9]         [7,8,9]         3
            H   ({b=>"c"})         {b=>"c"}        [{b=>"c"}]      {b=>"c"}
            I   ([1])              [1]             [[1]]           [1]
            J   ([1],[2])          [[1],[2]]       [[1],[2]]       [2]
            K   [7,8,9]            [7,8,9]         [[7,8,9]]       [7,8,9]
            L   (undef, "foo")     die "foo"       [undef, "foo"]  "foo"
            M   wantarray?1:0      1               [1]             0

        Cases F, H, I and M are common sticking points of the smart context
        in TT2. Note that list context always returns an arrayref from a
        method or function call. Smart context can give confusing results
        sometimes, especially the I and J cases. Case L for smart match is
        very surprising.

        The list and item context provide another feature for method calls.
        In smart context, TT will look for a hash key in the object by the
        same name as the method, if a method by that name doesn't exist. In
        item and list context Alloy will die if a method by that name cannot
        be found.

        The CALL_CONTEXT configuration item can be passed to new or it may
        also be set during runtime using the CONFIG directive. The following
        method call would be in list context:

            [% CONFIG CALL_CONTEXT => 'list';
               results = my_obj.get_results;
               CONFIG CALL_CONTEXT => 'smart'
            %]

        Note that we needed to restore CALL_CONTEXT to the default 'smart'
        value. Template::Alloy has added the "@()" (list) and the "$()"
        (item) context specifiers. The previous example could be written as:

            [% results = @( my_obj.get_results ) %]

        To call that same method in item (scalar) context you would do the
        following:

            [% results = $( my_obj.get_results ) %]

        The "@()" and "$()" operators are based on the Perl 6 counterpart.

    COMPILE_DIR
        Base directory to store compiled templates. Default undef. Compiled
        templates will only be stored if one of COMPILE_DIR and COMPILE_EXT
        is set.

        If set, the AST of parsed documents will be cached. If COMPILE_PERL
        is set, the compiled perl code will also be stored.

    COMPILE_EXT
        Extension to add to stored compiled template filenames. Default
        undef.

        If set, the AST of parsed documents will be cached. If COMPILE_PERL
        is set, the compiled perl code will also be stored.

    COMPILE_JS
        Default false.

        Requires installation of Template::Alloy::JS. When enabled, the
        parsed templates will be translated into Javascript and executed
        using the V8 javascript engine. If compile_dir is also set, this
        compiled javascript will be cached to disk.

        If your templates are short, there is little benefit to using this
        other than you can then use the JS directive. If your templates are
        long or you are running in a cached environment, this will speed up
        your templates.

        Certain limitations exist when COMPILE_JS is set, most notably the
        USE and VIEW directives are not supported, and method calls on
        objects passed to the template do not work (code refs passed in do
        work however). These limitations are due to the nature of
        JavaScript::V8 bind and Perl/JavaScript OO differences.

    COMPILE_PERL
        Default false.

        If set to 1 or 2, will translate the normal AST into a perl 5 code
        document. This document can then be executed directly, cached in
        memory, or cached on the file system depending upon the
        configuration items set.

        If set to 1, a perl code document will always be generated.

        If set to 2, a perl code document will only be generated if an AST
        has already been cached for the document. This should give a speed
        benefit and avoid extra compilation unless the document has been
        used more than once.

        If Alloy is running in a cached environment such as mod_perl, then
        using compile_perl can offer some speed benefit and makes Alloy
        faster than Text::Tmpl and as fast as HTML::Template::Compiled (but
        Alloy has more features).

        If you are not running in a cached environment, such as from
        commandline, or from CGI, it is generally faster to only run from
        the AST (with COMPILE_PERL => 0).

    CONSTANTS
        Hashref. Used to define variables that will be "folded" into the
        compiled template. Variables defined here cannot be overridden.

            CONSTANTS => {my_constant => 42},

            A template containing:

            [% constants.my_constant %]

            Will have the value 42 compiled in.

        Constants defined in this way can be chained as in [%
        constant.foo.bar.baz %].

    CONSTANT_NAMESPACE
        Allow for setting the top level of values passed in CONSTANTS.
        Default value is 'constants'.

    DEBUG
        Takes a list of constants |'ed together which enables different
        debugging modes. Alternately the lowercase names may be used
        (multiple values joined by a ",").

            The only supported TT values are:
            DEBUG_UNDEF (2)    - debug when an undefined value is used (now easier to use STRICT)
            DEBUG_DIRS  (8)    - debug when a directive is used.
            DEBUG_ALL   (2047) - turn on all debugging.

            Either of the following would turn on undef and directive debugging:

            DEBUG => 'undef, dirs',            # preferred
            DEBUG => 2 | 8,
            DEBUG => DEBUG_UNDEF | DEBUG_DIRS, # constants from Template::Constants

    DEBUG_FORMAT
        Change the format of messages inserted when DEBUG has DEBUG_DIRS set
        on. This essentially the same thing as setting the format using the
        DEBUG directive.

    DEFAULT
        The name of a default template file to use if the passed one is not
        found.

    DELIMITER
        String to use to split INCLUDE_PATH with. Default is :. It is more
        straight forward to just send INCLUDE_PATH an arrayref of paths.

    DUMP
        Configures the behavior of the DUMP tag. May be set to 0, a hashref,
        or another true value. Default is true.

        If set to 0, all DUMP directives will do nothing. This is useful if
        you would like to turn off the DUMP directives under some
        environments.

        IF set to a true value (or undefined) then DUMP directives will
        operate.

        If set to a hashref, the values of the hash can be used to configure
        the operation of the DUMP directives. The following are the values
        that can be set in this hash.

        EntireStash
            Default 1. If set to 0, then the DUMP directive will not print
            the entire contents of the stash when a DUMP directive is called
            without arguments.

        handler
            Defaults to an internal coderef. If set to a coderef, the DUMP
            directive will pass the arguments to be dumped and expects a
            string with the dumped data. This gives complete control over
            the dump process.

            Note 1: The default handler makes sure that values matching the
            private variable regex are not included. If you install your own
            handler, you will need to take care of these variables if you
            intend for them to not be shown.

            Note 2: If you would like the name of the variable to be dumped,
            include the string '$VAR1' and the DUMP directive will
            interpolate the value. For example, to dump all output as YAML -
            you could do the following:

                DUMP => {
                   handler => sub {
                       require YAML;
                       return "\$VAR1 =\n".YAML::Dump(shift);
                   },
                }

        header
            Default 1. Controls whether a header is printed for each DUMP
            directive. The header contains the file and line number the DUMP
            directive was called from. If set to 0 the headers are disabled.

        html
            Defaults to 1 if $ENV{'REQUEST_METHOD'} is set - 0 otherwise. If
            set to 1, then the output of the DUMP directive is passed to the
            html filter and encased in "pre" tags. If set to 0 no html
            encoding takes place.

        Sortkeys, Useqq, Ident, Pad, etc
            Any of the Data::Dumper configuration items may be passed.

    ENCODING
        Default undef. If set, and if Perl version is greater than or equal
        to 5.7.3 (when Encode.pm was first included), then Encode::decode
        will be called every time a template file is processed and will be
        passed the value of ENCODING and text from the template.

        This item can also be set using [% CONFIG ENCODING => encoding %]
        before calling INCLUDE or PROCESS directives to change encodings on
        the fly.

    END_TAG
        Set a string to use as the closing delimiter for TT. Default is
        "%]".

    ERROR
        Used as a fall back when the processing of a template fails. May
        either be a single filename that will be used in all cases, or may
        be a hashref of options where the keynames represent error types
        that will be handled by the filename in their value. A key named
        default will be used if no other matching keyname can be found. The
        selection process is similar to that of the TRY/CATCH/THROW
        directives (see those directives for more information).

            my $t = Template::Alloy->new({
                ERROR => 'general/catch_all_errors.html',
            });

            my $t = Template::Alloy->new({
                ERROR => {
                    default   => 'general/catch_all_errors.html',
                    foo       => 'catch_all_general_foo_errors.html',
                    'foo.bar' => 'catch_foo_bar_errors.html',
                },
            });

        Note that the ERROR handler will only be used for errors during the
        processing of the main document. It will not catch errors that occur
        in templates found in the PRE_PROCESS, POST_PROCESS, and WRAPPER
        configuration items.

    ERRORS
        Same as the ERROR configuration item. Both may be used
        interchangeably.

    EVAL_PERL
        Boolean. Default false. If set to a true value, PERL and RAWPERL
        blocks will be allowed to run. This is a potential security hole, as
        arbitrary perl can be included in the template. If Template::Toolkit
        is installed, a true EVAL_PERL value also allows the perl and
        evalperl filters to be used.

    FILTERS
        Allow for passing in TT style filters.

            my $filters = {
                filter1 =>  sub { my $str = shift; $s =~ s/./1/gs; $s },
                filter2 => [sub { my $str = shift; $s =~ s/./2/gs; $s }, 0],
                filter3 => [sub { my ($context, @args) = @_; return sub { my $s = shift; $s =~ s/./3/gs; $s } }, 1],
            };

            my $str = q{
                [% a = "Hello" %]
                1 ([% a | filter1 %])
                2 ([% a | filter2 %])
                3 ([% a | filter3 %])
            };

            my $obj = Template::Alloy->new(FILTERS => $filters);
            $obj->process(\$str) || die $obj->error;

        Would print:

                1 (11111)
                2 (22222)
                3 (33333)

        Filters passed in as an arrayref should contain a coderef and a
        value indicating if they are dynamic or static (true meaning
        dynamic). The dynamic filters are passed the pseudo context object
        and any arguments and should return a coderef that will be called as
        the filter. The filter coderef is then passed the string.

    GLOBAL_CACHE
        Default 0. If true, documents will be cached in
        $Template::Alloy::GLOBAL_CACHE. It may also be passed a hashref, in
        which case the documents will be cached in the passed hashref.

        The TT, Tmpl, and velocity will automatically cache documents in the
        object. The HTML::Template interface uses a new object each time.
        Setting the HTML::Template's CACHE configuration is the same as
        setting GLOBAL_CACHE.

    INCLUDE_PATH
        A string or an arrayref or coderef that returns an arrayref that
        contains directories to look for files included by processed
        templates. Defaults to "." (the current directory).

    INCLUDE_PATHS
        Non-TT item. Same as INCLUDE_PATH but only takes an arrayref. If not
        specified then INCLUDE_PATH is turned into an arrayref and stored in
        INCLUDE_PATHS. Overrides INCLUDE_PATH.

    INTERPOLATE
        Boolean. Specifies whether variables in text portions of the
        template will be interpolated. For example, the $variable and
        ${var.value} would be substituted with the appropriate values from
        the variable cache (if INTERPOLATE is on).

            [% IF 1 %]The variable $variable had a value ${var.value}[% END %]

    LOAD_PERL
        Indicates if the USE directive can fall back and try and load a perl
        module if the indicated module was not found in the PLUGIN_BASE
        path. See the USE directive. This configuration has no bearing on
        the COMPILE_PERL directive used to indicate using compiled perl
        documents.

    MAX_EVAL_RECURSE (Alloy only)
        Will use $Template::Alloy::MAX_EVAL_RECURSE if not present. Default
        is 50. Prevents runaway on the following:

            [% f = "[% f|eval %]" %][% f|eval %]

    MAX_MACRO_RECURSE (Alloy only)
        Will use $Template::Alloy::MAX_MACRO_RECURSE if not present. Default
        is 50. Prevents runaway on the following:

            [% MACRO f BLOCK %][% f %][% END %][% f %]

    NAMESPACE
        No Template::Namespace::Constants support. Hashref of hashrefs
        representing constants that will be folded into the template at
        compile time.

            Template::Alloy->new(NAMESPACE => {constants => {
                 foo => 'bar',
            }});

        Is the same as

            Template::Alloy->new(CONSTANTS => {
                 foo => 'bar',
            });

        Any number of hashes can be added to the NAMESPACE hash.

    NEGATIVE_STAT_TTL (Not in TT)
        Defaults to STAT_TTL which defaults to $STAT_TTL which defaults to
        1.

        Similar to STAT_TTL - but represents the time-to-live seconds until
        a document that was not found is checked again against the system
        for modifications. Setting this number higher will allow for fewer
        file system accesses. Setting it to a negative number will allow for
        the file system to be checked every hit.

    NO_INCLUDES
        Default false. If true, calls to INCLUDE, PROCESS, WRAPPER and
        INSERT will fail. This option is also available when using the
        process method.

    OUTPUT
        Alternate way of passing in the output location for processed
        templates. If process is not passed an output argument, it will look
        for this value.

        See the process method for a listing of possible values.

    OUTPUT_PATH
        Base path for files written out via the process method or via the
        redirect and file filters. See the redirect virtual method and the
        process method for more information.

    PLUGINS
        A hashref of mappings of plugin modules.

           PLUGINS => {
              Iterator => 'Template::Plugin::Iterator',
              DBI      => 'MyDBI',
           },

        See the USE directive for more information.

    PLUGIN_BASE
        Default value is Template::Plugin. The base module namespace that
        template plugins will be looked for. See the USE directive for more
        information. May be either a single namespace, or an arrayref of
        namespaces.

    POST_CHOMP
        Set the type of chomping at the ending of a tag. See the section on
        chomping for more information.

    POST_PROCESS
        Only available via when using the process interface.

        A list of templates to be processed and appended to the content
        after the main template. During this processing the "template"
        namespace will contain the name of the main file being processed.

        This is useful for adding a global footer to all templates.

    PRE_CHOMP
        Set the type of chomping at the beginning of a tag. See the section
        on chomping for more information.

    PRE_DEFINE
        Same as the VARIABLES configuration item.

    PRE_PROCESS
        Only available via when using the process interface.

        A list of templates to be processed before and pre-pended to the
        content before the main template. During this processing the
        "template" namespace will contain the name of the main file being
        processed.

        This is useful for adding a global header to all templates.

    PROCESS
        Only available via when using the process interface.

        Specify a file to use as the template rather than the one passed in
        to the ->process method.

    RECURSION
        Boolean. Default false. Indicates that INCLUDED or PROCESSED files
        can refer to each other in a circular manner. Be careful about
        recursion.

    RELATIVE
        Boolean. Default false. If true, allows filenames to be specified
        that are relative to the currently running process.

    SEMICOLONS
        Boolean. Default false. If true, then the syntax will require that
        semi-colons separate multiple directives in the same tag. This is
        useful for keeping the syntax a little more clean as well as trouble
        shooting some errors.

    SHOW_UNDEFINED_INTERP (Not in TT)
        Default false (default true in Velocity). If INTERPOLATE is true,
        interpolated dollar variables that return undef will be removed.
        With SHOW_UNDEFINED_INTERP set, undef values will leave the variable
        there.

            [% CONFIG INTERPOLATE => 1 %]
            [% SET foo = 1 %][% SET bar %]
            ($foo)($bar) ($!foo)($!bar)

        Would print:

            (1)() (1)()

        But the following:

            [% CONFIG INTERPOLATE => 1, SHOW_UNDEFINED_INTERP => 1 %]
            [% SET foo = 1 %][% SET bar %]
            ($foo)($bar) ($!foo)($!bar)

        Would print:

            (1)($bar) (1)()

        Note that you can use an exclamation point directly after the dollar
        to make the variable silent. This is similar to how Velocity works.

    START_TAG
        Set a string or regular expression to use as the opening delimiter
        for TT. Default is "[%". You should be sure that the tag does not
        include grouping parens or INTERPOLATE will not function properly.

    STASH
        Template::Alloy manages its own stash of variables. You can pass a
        Template::Stash or Template::Stash::XS object, but Template::Alloy
        will copy all of values out of the object into its own stash.
        Template::Alloy won't use any of the methods of the passed STASH
        object. The STASH option is only available when using the process
        method.

    STAT_TTL
        Defaults to $STAT_TTL which defaults to 1. Represents time-to-live
        seconds until a cached in memory document is compared to the file
        system for modifications. Setting this number higher will allow for
        fewer file system accesses. Setting it to a negative number will
        allow for the file system to be checked every hit.

    STREAM
        Defaults to false. If set to true, generated template content will
        be printed to the currently selected filehandle (default is STDOUT)
        as soon as it is ready - there will be no buffering of the output.

        The Stream role uses the Play role's directives (non-compiled_perl).

        All directives and configuration work, except for the following
        exceptions:

        CLEAR directive
            Because the output is not buffered - the CLEAR directive would
            have no effect. The CLEAR directive will throw an error when
            STREAM is on.

        TRIM configuration
            Because the output is not buffered - trim operations cannot be
            played on the output buffers.

        WRAPPER configuration/directive
            The WRAPPER configuration and directive items effectively turn
            off STREAM since the WRAPPERS are generated in reverse order and
            because the content is inserted into the middle of the WRAPPERS.
            WRAPPERS will still work, they just won't stream.

        VARIOUS errors
            Because the template is streaming, items that cause errors my
            result in partially printed pages - since the error would occur
            part way through the print.

        All output is printed directly to the currently selected filehandle
        (defaults to STDOUT) via the CORE::print function. Any output
        parameter passed to process or process_simple will be ignored.

        If you would like the output to go to another handle, you will need
        to select that handle, process the template, and re-select STDOUT.

    STRICT
        Defaults to false. If set to true, any undefined variable that is
        encountered will cause the processing of the template to abort. This
        can be caught with a TRY block. This can be useful for making sure
        that the template only attempts to use variables that were correctly
        initialized similar in spirit to Perl's "use strict."

        When this occurs the strict_throw method is called.

        See the STRICT_THROW configuration for additional options.

        Similar functionality could be implemented using UNDEFINED_ANY.

        The STRICT configuration item can be passed to new or it may also be
        set during runtime using the CONFIG directive. Once set though it
        cannot be disabled for the duration of the current template and sub
        components. For example you could call [% CONFIG STRICT => 1 %] in
        header.tt and strict mode would be enabled for the header.tt and any
        sub templates processed by header.tt.

    STRICT_THROW (not in TT)
        Default undef. Can be set to a subroutine which will be called when
        STRICT is set and an undefined variable is processed. It will be
        passed the error type, error message, and a hashref of template
        information containing the current component being processed, the
        current outer template being processed, the identity reference for
        the variable, and the stringified name of the identity. This
        override can be used for filtering allowable elements.

            my $ta = Template::Alloy->new({
                STRICT => 1,
                STRICT_THROW => sub {
                    my ($ta, $err_type, $msg, $args) = @_;

                    return if $args->{'component'} eq 'header.tt'
                              && $args->{'template'} eq 'main.html'
                              && $args->{'name'} eq 'foo.bar(1)'; # stringified identity name

                    $ta->throw($err_type, $msg); # all other undefined variables die
                },
            });

    SYNTAX (not in TT)
        Defaults to "cet". Indicates the syntax that will be used for
        parsing included templates or eval'ed strings. You can use the
        CONFIG directive to change the SYNTAX on the fly (it will not affect
        the syntax of the document currently being parsed).

        The syntax may be passed in upper or lower case.

        The available choices are:

            alloy - Template::Alloy style - the same as TT3
            tt3   - Template::Toolkit ver3 - same as Alloy
            tt2   - Template::Toolkit ver2 - almost the same as TT3
            tt1   - Template::Toolkit ver1 - almost the same as TT2
            ht    - HTML::Template - same as HTML::Template::Expr without EXPR
            hte   - HTML::Template::Expr
            js    - JavaScript style - requires compile_js to be set.
            jsr   - JavaScript Raw style - requires compile_js to be set.

        Passing in a different syntax allows for the process method to use a
        non-TT syntax and for the output method to use a non-HT syntax.

        The following is a sample of HTML::Template interface usage parsing
        a Template::Toolkit style document.

            my $obj = Template::Alloy->new(filename => 'my/template.tt'
                                             syntax   => 'cet');
            $obj->param(\%swap);
            print $obj->output;

        The following is a sample of Template::Toolkit interface usage
        parsing a HTML::Template::Expr style document.

            my $obj = Template::Alloy->new(SYNTAX => 'hte');
            $obj->process('my/template.ht', \%swap);

        You can use the define_syntax method to add another custom syntax to
        the list of available options.

    TAG_STYLE
        Allow for setting the type of tag delimiters to use for parsing the
        TT. See the TAGS directive for a listing of the available types.

    TRIM
        Remove leading and trailing whitespace from blocks and templates.
        This operation is performed after all enclosed template tags have
        been executed.

    UNDEFINED_ANY
        This is not a TT configuration option. This option expects to be a
        code ref that will be called if a variable is undefined during a
        call to play_expr. It is passed the variable identity array as a
        single argument. This is most similar to the "undefined" method of
        Template::Stash. It allows for the "auto-defining" of a variable for
        use in the template. It is suggested that UNDEFINED_GET be used
        instead as UNDEFINED_ANY is a little to general in defining
        variables.

        You can also sub class the module and override the undefined_any
        method.

    UNDEFINED_GET
        This is not a TT configuration option. This option expects to be a
        code ref that will be called if a variable is undefined during a
        call to GET. It is passed the variable identity array as a single
        argument. This is more useful than UNDEFINED_ANY in that it is only
        called during a GET directive rather than in embedded expressions
        (such as [% a || b || c %]).

        You can also sub class the module and override the undefined_get
        method.

    V1DOLLAR
        This allows for some compatibility with TT1 templates. The only real
        behavior change is that [% $foo %] becomes the same as [% foo %].
        The following is a basic table of changes invoked by using V1DOLLAR.

           With V1DOLLAR        Equivalent Without V1DOLLAR (Normal default)
           "[% foo %]"          "[% foo %]"
           "[% $foo %]"         "[% foo %]"
           "[% ${foo} %]"       "[% ${foo} %]"
           "[% foo.$bar %]"     "[% foo.bar %]"
           "[% ${foo.bar} %]"   "[% ${foo.bar} %]"
           "[% ${foo.$bar} %]"  "[% ${foo.bar} %]"
           "Text: $foo"         "Text: $foo"
           "Text: ${foo}"       "Text: ${foo}"
           "Text: ${$foo}"      "Text: ${foo}"

    V2EQUALS
        Default 1 in the TT syntax, defaults to 0 in the HTML::Template
        syntax.

        If set to 1 then "==" is an alias for "eq" and "!= is an alias for
        "ne".

            [% CONFIG V2EQUALS => 1 %][% ('7' == '7.0') || 0 %]
            [% CONFIG V2EQUALS => 0 %][% ('7' == '7.0') || 0 %]

            Prints

            0
            1

    V2PIPE
        Restores the behavior of the pipe operator to be compatible with
        TT2.

        With V2PIPE = 1

            [%- BLOCK a %]b is [% b %]
            [% END %]
            [%- PROCESS a b => 237 | repeat(2) %]

            # output of block "a" with b set to 237 is passed to the repeat(2) filter

            b is 237
            b is 237

        With V2PIPE = 0 (default)

            [%- BLOCK a %]b is [% b %]
            [% END %]
            [% PROCESS a b => 237 | repeat(2) %]

            # b set to 237 repeated twice, and b passed to block "a"

            b is 237237

    VARIABLES
        A hashref of variables to initialize the template stash with. These
        variables are available for use in any of the executed templates.
        See the section on VARIABLES for the types of information that can
        be passed in.

    VMETHOD_FUNCTIONS
        Defaults to 1. All scalar virtual methods are available as top level
        functions as well. This is not true of TT2. In Template::Alloy the
        following are equivalent:

            [% "abc".length %]
            [% length("abc") %]

        You may set VMETHOD_FUNCTIONS to 0 to disable this behavior.

    WRAPPER
        Only available via when using the process interface.

        Operates similar to the WRAPPER directive. The option can be given a
        single filename, or an arrayref of filenames that will be used to
        wrap the processed content. If an arrayref is passed the filenames
        are processed in reverse order, so that the first filename specified
        will end up being on the outside (surrounding all other wrappers).

           my $t = Template::Alloy->new(
               WRAPPER => ['my/wrappers/outer.html', 'my/wrappers/inner.html'],
           );

        Content generated by the PRE_PROCESS and POST_PROCESS will come
        before and after (respectively) the content generated by the WRAPPER
        configuration item.

        See the WRAPPER directive for more examples of how wrappers are
        constructed.

CONFIGURATION (HTML::Template STYLE)
    The following HTML::Template and HTML::Template::Expr configuration
    variables are supported (in HTML::Template documentation order). Note:
    for further discussion you can refer to the HT documentation. Many of
    the variables mentioned in the TT CONFIGURATION section apply here as
    well. Unless noted, these items only apply when using the output method.

    Items may be passed in upper or lower case. All passed items are
    resolved to upper case.

    These variables should be passed to the "new" constructor.

        my $obj = Template::Alloy->new(
            type   => 'filename',
            source => 'my/template.ht',
            die_on_bad_params => 1,
            loop_context_vars => 1,
            global_vars       => 1
            post_chomp => "=",
            pre_chomp  => "-",
        );

    TYPE
        Can be one of filename, filehandle, arrayref, or scalarref.
        Indicates what type of input is in the "source" configuration item.

    SOURCE
        Stores where to read the input file. The type is specified in the
        "type" configuration item.

    FILENAME
        Indicates a filename to read the template from. Same as putting the
        filename in the "source" item and setting "type" to "filename".

        Must be set to enable caching.

    FILEHANDLE
        Should contain an open filehandle to read the template from. Same as
        putting the filehandle in the "source" item and setting "type" to
        "filehandle".

        Will not be cached.

    ARRAYREF
        Should contain an arrayref whose values are the lines of the
        template. Same as putting the arrayref in the "source" item and
        setting "type" to "arrayref".

        Will not be cached.

    SCALARREF
        Should contain an reference to a scalar that contains the template.
        Same as putting the scalar ref in the "source" item and setting
        "type" to "scalarref".

        Will not be cached.

    CACHE
        If set to one, then Alloy will use a global, in-memory document
        cache to store compiled templates in between calls. This is
        generally only useful in a mod_perl environment. The document is
        checked for a different modification time at each request.

    BLIND_CACHE
        Same as with cache enabled, but will not check if the document has
        been modified.

    FILE_CACHE
        If set to 1, will cache the compiled document on the file system. If
        true, file_cache_dir must be set.

    FILE_CACHE_DIR
        The directory where to store cached documents when file_cache is
        true. This is similar to the TT compile_dir option.

    DOUBLE_FILE_CACHE
        Uses a combination of file_cache and cache.

    PATH
        Same as INCLUDE_PATH when using the process method.

    ASSOCIATE
        May be a single CGI object or an arrayref of objects. The params
        from these objects will be added to the params during the output
        call.

    CASE_SENSITIVE
        Allow passed variables set through the param method, or the
        associate configuration to be used case sensitively. Default is off.
        It is highly suggested that this be set to 1.

    LOOP_CONTEXT_VARS
        Default false. When true, calls to the loop directive will create
        the following variables that give information about the current
        iteration of the loop:

           __first__   - True on first iteration only
           __last__    - True on last iteration only
           __inner__   - True on any iteration that isn't first or last
           __odd__     - True on odd iterations
           __counter__ - The iteration count

        These variables are also available to LOOPs run under TT syntax if
        loop_context_vars is set and if QR_PRIVATE is set to 0.

    GLOBAL_VARS.
        Default true in HTE mode. Default false in HT. Allows top level
        variables to be used in LOOPs. When false, only variables defined in
        the current LOOP iteration hashref will be available.

    DEFAULT_ESCAPE
        Controls the type of escape used on named variables in TMPL_VAR
        directives. Can be one of HTML, URL, or JS. The values of TMPL_VAR
        directives will be encoded with this type unless they specify their
        own type via an ESCAPE attribute.

        You may alternately use the AUTO_FILTER directive which can be any
        of the item vmethod filters (you must use lower case when specifying
        the AUTO_FILTER directive). The AUTO_FILTER directive will also be
        applied to TMPL_VAR EXPR and TMPL_GET items while DEFAULT_ESCAPE
        only applies to TMPL_VAR NAME items.

    NO_TT
        Default false in 'hte' syntax. Default true in 'ht' syntax. If true,
        no extended TT directives will be allowed.

        The output method uses 'hte' syntax by default.

SEMI PUBLIC METHODS
    The following list of methods are other interesting methods of Alloy
    that may be re-implemented by subclasses of Alloy.

    "exception"
        Creates an exception object blessed into the package listed in
        Template::Alloy::Exception.

    "execute_tree"
        Executes a parsed tree (returned from parse_tree)

    "play_expr"
        Play the parsed expression. Turns a variable identity array into the
        parsed variable. This method is also responsible for playing
        operators and running virtual methods and filters. The variable
        identity array may also contain literal values, or operator identity
        arrays.

    "include_filename"
        Takes a file path, and resolves it into the full filename using
        paths from INCLUDE_PATH or INCLUDE_PATHS.

    "_insert"
        Resolves the file passed, and then returns its contents.

    "list_filters"
        Dynamically loads the filters list from Template::Filters when a
        filter is used that does not have a native implementation in Alloy.

    "load_template"
        Given a filename or a string reference will return a "document"
        hashref hash that contains the parsed tree.

            my $doc = $self->load_template($file); # errors die

        This method handles the in-memory caching of the document.

    "load_tree"
        Given the "document" hashref, will either load the parsed AST from
        file (if configured to do so), or will load the content, parse the
        content using the Parse role, and will return the tree. File based
        caching of the parsed AST happens here.

    "load_perl"
        Only used if COMPILE_PERL is true (default is false).

        Given the "document" hashref, will either load the compiled perl
        from file (if configured to do so), or will load the AST using
        "load_tree", will compile a new perl code document using the Compile
        role, and will return the perl code. File based caching of the
        compiled perl happens here.

    "parse_tree"
        Parses the passed string ref with the appropriate template syntax.

        See Template::Alloy::Parse for more details.

    "parse_expr"
        Parses the passed string ref for a variable or expression.

        See Template::Alloy::Parse for more details.

    "parse_args"
        See Template::Alloy::Parse for more details.

    "set_variable"
        Used to set a variable. Expects a variable identity array and the
        value to set. It will autovifiy as necessary.

    "strict_throw"
        Called during processing of template when STRICT configuration is
        set and an uninitialized variable is met. Arguments are the variable
        identity reference. Will call STRICT_THROW configuration item if
        set, otherwise will call throw with a useful message.

    "throw"
        Creates an exception object from the arguments and dies.

    "undefined_any"
        Called during play_expr if a value is returned that is undefined.
        This could be used to magically create variables on the fly. This is
        similar to Template::Stash::undefined. It is suggested that
        undefined_get be used instead. Default behavior returns undef. You
        may also pass a coderef via the UNDEFINED_ANY configuration
        variable. Also, you can try using the DEBUG => 'undef',
        configuration option which will throw an error on undefined
        variables.

    "undefined_get"
        Called when a variable is undefined during a GET directive. This is
        useful to see if a value that is about to get inserted into the text
        is undefined. undefined_any is a little too general for most cases.
        Also, you may pass a coderef via the UNDEFINED_GET configuration
        variable.

OTHER UTILITY METHODS
    The following is a brief list of other methods used by Alloy. Generally,
    these shouldn't be overwritten by subclasses.

    "ast_string"
        Returns perl code representation of a variable.

    "context"
        Used to create a "pseudo" context object that allows for portability
        of TT plugins, filters, and perl blocks that need a context object.
        Uses the Template::Alloy::Context class.

    "debug_node"
        Used to get debug info on a directive if DEBUG_DIRS is set.

    "get_line_number_by_index"
        Used to turn string index position into line number

    "interpolate_node"
        Used for parsing text nodes for dollar variables when interpolate is
        on.

    "play_operator"
        Provided by the Operator role. Allows for playing an operator AST.

        See Template::Alloy::Operator for more details.

    "apply_precedence"
        Provided by the Parse role. Allows for parsed operator array to be
        translated to a tree based upon operator precedence.

    "_process"
        Called by process and the PROCESS, INCLUDE and other directives.

    "slurp"
        Reads contents of passed filename - throws file exception on error.

    "split_paths"
        Used to split INCLUDE_PATH or other directives if an arrayref is not
        passed.

    "tt_var_string"
        Returns a template toolkit representation of a variable.

    "_vars"
        Return a reference to the current stash of variables. This is
        currently only used by the pseudo context object and may disappear
        at some point.

THANKS
    Thanks to Andy Wardley for creating Template::Toolkit.

    Thanks to Sam Tregar for creating HTML::Template.

    Thanks to David Lowe for creating Text::Tmpl.

    Thanks to the Apache Velocity guys.

    Thanks to Ben Grimm for a patch to allow passing a parsed document to
    the ->process method.

    Thanks to David Warring for finding a parse error in HTE syntax.

    Thanks to Carl Franks for adding the base ENCODING support.

AUTHOR
    Paul Seamons <paul@seamons.com>

LICENSE
    This module may be distributed under the same terms as Perl itself.

