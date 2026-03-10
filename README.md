# Perl Best Practices Guidelines


## Table of Contents

1. [Introduction & Philosophy](#1-introduction--philosophy)
2. [Code Style & Formatting](#2-code-style--formatting)
3. [Core Language Practices](#3-core-language-practices)
4. [Subroutines & Functions](#4-subroutines--functions)
5. [Variables & Data Structures](#5-variables--data-structures)
6. [Error Handling](#6-error-handling)
7. [Input Validation & Security](#7-input-validation--security)
8. [Testing Best Practices](#8-testing-best-practices)
9. [Documentation](#9-documentation)
10. [Module Development](#10-module-development)
11. [Performance Considerations](#11-performance-considerations)
12. [Code Quality Tools](#12-code-quality-tools)
13. [Common Anti-Patterns](#13-common-anti-patterns)
14. [Resources & References](#14-resources--references)

---

## 1. Introduction & Philosophy

## Purpose and Scope

This guidelines document establishes best practices for Perl development across our projects. These practices aim to:

- **Improve code readability** and maintainability
- **Reduce bugs** through proven patterns
- **Enhance team collaboration** with consistent standards
- **Leverage modern Perl features** (v5.36+)
- **Align with industry standards** and community recommendations

## Target Audience

These guidelines are designed for:

- **Junior developers** beginning their Perl journey (with references to foundational concepts)
- **Intermediate developers** seeking to refine their practices
- **Senior developers** establishing team standards
- **Code reviewers** evaluating pull requests
- **DevOps engineers** maintaining Perl scripts and automation

## How to Use This Document

- **Quick reference**: Jump to sections relevant to your current task
- **Deep learning**: Read sections sequentially for foundational understanding
- **Team adoption**: Use checklists and anti-patterns for code reviews
- **Tool setup**: Reference configuration sections for IDE and linting tools
- **Pattern lookup**: Search for specific patterns (e.g., "error handling", "testing")

## Core Philosophy: Readability Over Cleverness

> **"Make your code clear first, clever second."**

### Key Principles

1. **Explicit is better than implicit**
   - State intentions clearly
   - Avoid cryptic shortcuts
   - Use meaningful variable names

2. **Maintainability over performance micro-optimizations**
   - Focus optimization efforts on bottlenecks identified through profiling
   - Clear code is easier to optimize later
   - Premature optimization leads to unmaintainable code

3. **Fail fast and loudly**
   - Use `strict` and `warnings` (mandatory)
   - Validate input early
   - Report errors with context

4. **The Perl Philosophy**
   - TMTOWTDI (There's More Than One Way To Do It)
   - BUT: Choose the clearest way for your team
   - Document your choices

### Example: Philosophy in Practice

```perl
# ❌ BAD - Clever but obscure
my @nums = (1..10);
my $sum = eval { my $s; $s += $_ for @nums; $s };

# ✅ GOOD - Clear intent
my @nums = (1..10);
my $sum = 0;
for my $num (@nums) {
    $sum += $num;
}

# ✅ BETTER - Use appropriate tools
use List::Util qw(sum);
my @nums = (1..10);
my $sum = sum @nums;
```

### The Readability Checklist

Before considering code "done":

- [ ] Someone unfamiliar with the code can understand it in 5 minutes
- [ ] Variable names clearly describe their purpose
- [ ] Complex logic is broken into clearly-named functions
- [ ] All external dependencies (CPAN modules) are documented
- [ ] Error cases are handled and reported clearly
- [ ] Comments explain *why*, not *what*
- [ ] Tests demonstrate expected behavior


[↑ Back to TOC](#table-of-contents)

---

## 2. Code Style & Formatting

## Indentation

**Standard: 4 spaces per indentation level**

- Use spaces, not tabs (enforced via `.editorconfig` and `perltidy`)
- Be consistent throughout your codebase
- Modern editors can auto-configure this

### Rationale

- Tab width varies across editors and environments
- Mixed tabs/spaces cause invisible formatting issues
- Industry standard for Perl development

### Example

```perl
# ✅ GOOD - 4 space indentation
sub process_data {
    my ($data) = @_;
    
    if (defined $data) {
        for my $item (@$data) {
            my $result = calculate($item);
            return $result if $result > 0;
        }
    }
    
    return undef;
}

# ❌ BAD - Inconsistent indentation
sub process_data {
my ($data) = @_;
if (defined $data) {
for my $item (@$data) {
my $result = calculate($item);
return $result if $result > 0;
}
}
return undef;
}
```

## Brace Placement

**Opening braces on the same line as the keyword**

```perl
# ✅ GOOD
if ($condition) {
    # code
} elsif ($other) {
    # code
} else {
    # code
}

sub my_function {
    # code
}

for my $item (@items) {
    # code
}

# ❌ BAD - Opening brace on next line
if ($condition)
{
    # code
}

sub my_function
{
    # code
}
```

**Exception**: When a conditional is complex, you may break before the brace:

```perl
# Acceptable for complex conditions
if ($self->can_process($data)
    && $self->validate_schema($schema)
    && $self->check_permissions($user)
) {
    # code
}
```

## Line Length

**Recommended: 80-100 characters per line**

- **Hard limit**: 120 characters
- Exceptions: URLs, long strings that can't be split logically
- Most modern IDE rulers can enforce this

### Rationale

- Readable on standard displays and terminals
- Fits on side-by-side diffs and splits
- Reduces cognitive load (longer lines are harder to parse)
- Terminal-friendly for remote access

### Breaking Long Lines

```perl
# ✅ GOOD - Break logical sections
my $config = {
    host     => $self->host,
    port     => $self->port,
    database => $self->database_name,
    username => $self->credentials->{user},
    password => $self->credentials->{pass},
};

# ✅ GOOD - Break long method chains
my $result = $data_obj
    ->validate()
    ->filter()
    ->transform()
    ->get_result();

# ✅ GOOD - Break arguments
call_function(
    $first_param,
    $second_param,
    $third_param,
    { option1 => 'value', option2 => 'other' }
);

# ❌ BAD - Single line too long
my $config = { host => $self->host, port => $self->port, database => $self->database_name, username => $self->credentials->{user}, password => $self->credentials->{pass} };
```

## Whitespace and Spacing

### Around Operators

```perl
# ✅ GOOD - Space around operators
my $result = $a + $b;
my $comparison = $x == $y;
my $string = "prefix_" . $suffix;
my @filtered = grep { $_ > 10 } @items;
my @mapped = map { $_->name } @users;

# ❌ BAD - No spaces around operators
my $result=$a+$b;
my $comparison=$x==$y;
my @filtered=grep{$_>10}@items;
```

### After Commas

```perl
# ✅ GOOD - Space after comma
my @list = (1, 2, 3, 4, 5);
call_function($arg1, $arg2, $arg3);
my $hash = { key1 => 'val', key2 => 'val' };

# ❌ BAD - No space after comma
my @list = (1,2,3,4,5);
call_function($arg1,$arg2,$arg3);
```

### Inside Hash/Array Literals

```perl
# ✅ GOOD - Aligned values
my %config = (
    host     => 'localhost',
    port     => 5432,
    user     => 'admin',
    password => 'secret',
);

# ✅ GOOD - Compact for simple structures
my @colors = qw(red green blue);
my %flags = (debug => 1, verbose => 0);

# Alignment improves readability for complex data
my @users = (
    { id => 1, name => 'Alice', email => 'alice@example.com' },
    { id => 2, name => 'Bob',   email => 'bob@example.com'   },
    { id => 3, name => 'Carol', email => 'carol@example.com' },
);
```

## Naming Conventions

**Consistent naming improves code clarity and prevents bugs.**

### Variables

```perl
# Scalars: $snake_case
my $user_name = 'john';
my $total_count = 42;
my $max_retries = 3;

# Arrays: @plural or @descriptive (treat as plural)
my @users = ();
my @active_sessions = ();
my @failed_checks = ();

# Hashes: %singular_descriptive (conceptually singular collection)
my %user_config = ();
my %error_messages = ();
my %cache_data = ();

# Regex matches
my $email_pattern = qr/[\w\.-]+@[\w\.-]+\.\w+/;
my @matches = ($string =~ /pattern/g);

# Boolean-like values (optional but clarifying)
my $is_valid = 1;
my $has_error = 0;
my $should_retry = 1;
```

### Subroutines and Functions

```perl
# ✅ GOOD - Descriptive lowercase with underscores
sub calculate_total_price { }
sub validate_email_address { }
sub fetch_user_from_database { }
sub process_queue_items { }

# ❌ BAD - Ambiguous or non-descriptive names
sub calc { }
sub get { }
sub do_stuff { }
sub process { }

# ❌ BAD - CamelCase in procedural code
sub CalculateTotalPrice { }
```

### Packages and Modules

```perl
# ✅ GOOD - CamelCase for modules
package MyApp::Controllers::UserController;
package MyApp::Models::User;
package MyApp::Utils::DateHelper;

# ❌ BAD - Lowercase package names
package myapp::controllers::usercontroller;
```

### Constants

```perl
# ✅ GOOD - Use constant pragma with lowercase names
use constant MAX_RETRIES => 3;
use constant DEFAULT_TIMEOUT => 30;
use constant VALID_STATUSES => [qw(active pending closed)];

# ✅ GOOD - Package-level constants with our and uppercase
our $VERSION = '1.0.0';
our $MAX_BUFFER_SIZE = 8192;

# Alternative: Readonly for stricter behavior
use Readonly;
Readonly my $API_KEY => $ENV{API_KEY};
```

### Temporary/Loop Variables

```perl
# ✅ GOOD - Even loop vars should be descriptive
for my $user (@users) {
    print $user->{name};
}

for my $i (0 .. $#array) {
    my $item = $array[$i];
}

# ❌ BAD - Using $_ when not necessary
for (@users) {
    print $_->{name};
}

# ✅ GOOD - Use $_ only in map/grep contexts where it's clear
my @names = map { $_->{name} } @users;
my @active = grep { $_->is_active } @users;
```

## Style Guide Checklist

Use this checklist for code reviews:

- [ ] Indentation is consistent 4 spaces
- [ ] Opening braces are on the same line as keywords
- [ ] Lines are under 100 characters (except URLs)
- [ ] Spaces around binary operators
- [ ] Spaces after commas
- [ ] Hash values are aligned for readability
- [ ] Variable names use `$snake_case`, `@plural_arrays`, `%singular_hashes`
- [ ] Subroutine names use lowercase with underscores
- [ ] Constants use UPPER_CASE or `use constant`
- [ ] No tab characters (spaces only)
- [ ] No trailing whitespace

## Tooling Support

### `.perltidyrc` Configuration

Create this file in your project root to auto-format code:

```ini
-i          # in-place editing
-b          # backup original
-l=100      # line length
-i=4        # indent size 4
-st         # use standard output
-nsfs       # no space before semicolon
-pt=2       # parenthesis token width
-bt=2       # bracket token width
-sbt=2      # square bracket token width
-bbt=2      # block brace token width
-nws        # no want to skip blank lines
```

### Editor Configuration (`.editorconfig`)

```ini
root = true

[*]
charset = utf-8
insert_final_newline = true
trim_trailing_whitespace = true

[*.pl]
indent_style = space
indent_size = 4
max_line_length = 100

[*.t]
indent_style = space
indent_size = 4
```


---


[↑ Back to TOC](#table-of-contents)

---

## 3. Core Language Practices

## Always Use `strict` and `warnings` (Mandatory)

These two pragmas are **non-negotiable** in every Perl script and module. They catch bugs early and enforce modern practices.

### `use strict`

Prevents common errors by:
- Requiring variables to be declared with `my`, `our`, or `local`
- Disallowing bareword filehandles
- Disallowing symbolic references (in most cases)

```perl
# ✅ GOOD - Always include strict
use strict;
use warnings;

my $name = 'Alice';           # ✅ OK - declared with my
print $name;

# ❌ BAD - Without strict (would fail)
$undeclared = 'Bob';          # ERROR: Global symbol "$undeclared" requires explicit package name

# ❌ BAD - Bareword filehandle (fails with strict)
open FILE, '<', 'data.txt';   # ERROR: Bareword "FILE" not allowed

# ✅ GOOD - Proper filehandle with strict
open my $fh, '<', 'data.txt' or die "Cannot open: $!";
```

### `use warnings`

Reports deprecations and likely errors:
- Uninitialized values in operations
- Undefined subroutine calls
- Redefined subroutines
- Arguments to built-in functions without required parameters

```perl
use strict;
use warnings;

# ⚠️ WARNING - Uninitialized value in addition
my $x;
my $result = $x + 5;    # Warning: Use of uninitialized value $x

# ⚠️ WARNING - Redefinition
sub greet { print "Hello\n"; }
sub greet { print "Hi\n"; }   # Warning: Subroutine greet redefined

# ✅ GOOD - Initialize before use
my $y = 0;
my $result = $y + 5;    # No warning
```

### Standard Header for All Code

```perl
#!/usr/bin/env perl
use strict;
use warnings;

# ... rest of code ...
```

## Modern Perl Features (v5.36+)

These features are stable, well-documented, and recommended for new code.

### Subroutine Signatures

**Available in Perl v5.20+, stable in v5.36+**

Cleaner, more explicit parameter handling than `my ($param1, $param2) = @_;`

```perl
# ✅ GOOD - Modern signatures (Perl 5.36+)
use feature 'signatures';
no warnings 'experimental::signatures';

sub add_numbers ($x, $y) {
    return $x + $y;
}

sub greet ($name, $title = 'Friend') {
    return "Hello, $title $name!";
}

# ✅ GOOD - Multiple parameters with defaults
sub configure_server ($host, $port = 8080, $ssl = 0) {
    # $host is required
    # $port defaults to 8080
    # $ssl defaults to 0
}

# ❌ OLD WAY - Still valid but less clear
sub add_numbers {
    my ($x, $y) = @_;
    return $x + $y;
}
```

### The `isa` Operator

**Available in Perl v5.32+**

Clean syntax for type checking:

```perl
use feature 'isa';

# ✅ GOOD - Modern type checking
if ($value isa 'ARRAY') {
    # Process array
}

if ($object isa 'DateTime') {
    # Process DateTime object
}

# ❌ OLD WAY - Still valid but more verbose
if (ref($value) eq 'ARRAY') {
    # Process array
}

if (ref($object) eq 'DateTime') {
    # Process DateTime object
}
```

### `defer` Blocks

**Available in Perl v5.36+**

Automatically executes code at end of scope (like Go's defer):

```perl
use feature 'defer';

# ✅ GOOD - Resource cleanup with defer
sub read_file ($filename) {
    open my $fh, '<', $filename or die "Cannot open: $!";
    defer { close $fh; }  # Automatically closes at end of function
    
    my $content = do { local $/; <$fh> };
    return $content;
}

# OLD WAY - Traditional approach
sub read_file_old {
    my ($filename) = @_;
    open my $fh, '<', $filename or die "Cannot open: $!";
    
    my $content = do { local $/; <$fh> };
    close $fh;
    return $content;
}
```

### Built-in Boolean Values

**Available in Perl v5.36+**

Cleaner boolean literals:

```perl
use feature 'builtin';
use builtin 'true', 'false';

# ✅ GOOD - Explicit booleans (Perl 5.36+)
my %flags = (
    debug   => true,
    verbose => false,
    logging => true,
);

# ✅ STILL GOOD - Traditional approach works everywhere
my %flags_old = (
    debug   => 1,
    verbose => 0,
    logging => 1,
);
```

## Features to Avoid (Obsolete/Deprecated)

These features are outdated or dangerous and should **not** be used in new code.

### Indirect Object Notation

```perl
# ❌ DANGEROUS - Indirect object notation
my $obj = new MyClass($param);
my $result = method $obj, $arg1, $arg2;

# ✅ GOOD - Direct method call
my $obj = MyClass->new($param);
my $result = $obj->method($arg1, $arg2);

# WHY? Indirect notation is ambiguous and can lead to unexpected behavior:
# - Bareword names might be confused with functions
# - Readability suffers
# - Refactoring becomes error-prone
```

### Multidimensional Array Emulation

```perl
# ❌ BAD - Multidimensional array emulation (outdated)
my @matrix;
$matrix[0][1] = 'value';     # Actually creates references
$matrix[0,1] = 'value';      # Dies with strict

# ✅ GOOD - Use array references
my @matrix;
$matrix[0] = [];
$matrix[0]->[1] = 'value';

# ✅ GOOD - Use nested data structures clearly
my @matrix = map { [] } 1..3;
$matrix[0]->[1] = 'value';

# ✅ BETTER - Use descriptive structures
my $data = {
    matrix => [
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],
    ],
};
```

### Bareword Filehandles

```perl
# ❌ BAD - Bareword filehandles
open FILE, '<', 'data.txt';
while (<FILE>) {
    print $_;
}
close FILE;

# ✅ GOOD - Lexical filehandles
open my $fh, '<', 'data.txt' or die "Cannot open: $!";
while (<$fh>) {
    print $_;
}
close $fh;
```

### Prototypes

Generally avoid subroutine prototypes. They don't work as expected in most cases:

```perl
# ❌ AVOID - Prototypes are confusing
sub process ($$) {
    my ($x, $y) = @_;
    return $x + $y;
}

# ✅ GOOD - Modern signatures or explicit parameter handling
sub process ($x, $y) {
    return $x + $y;
}

# Limited use case: Overriding built-in operators
use overload '+' => sub {
    my ($left, $right) = @_;
    # Custom addition
};
```

## Best Practices Summary

```perl
# ✅ COMPLETE MODERN PERL TEMPLATE
#!/usr/bin/env perl
use strict;
use warnings;
use feature 'signatures';
no warnings 'experimental::signatures';

use constant DEBUG => $ENV{DEBUG} // 0;

# Use modern syntax
sub process_data ($input, $options = {}) {
    return unless defined $input;
    
    # Explicit early returns for error conditions
    return undef if !$input;
    
    my $result = {};
    $result->{processed} = 1 if $options->{debug};
    
    return $result;
}

# Use lexical variables
my $config = { debug => 1 };

# Proper filehandle usage
open my $fh, '<', 'input.txt' or die "Cannot open: $!";
while (<$fh>) {
    chomp;
    # Process line
}
close $fh;

1;  # Module success indicator
```

## Core Language Practices Checklist

- [ ] All code starts with `use strict; use warnings;`
- [ ] No bareword filehandles (use lexical `my $fh`)
- [ ] No indirect object notation
- [ ] Modern signatures used for functions (Perl 5.20+)
- [ ] No prototypes (except for overloading)
- [ ] Variables declared with `my` (lexical scope)
- [ ] Data structures using references, not multidimensional array emulation
- [ ] All `open()` calls check return values
- [ ] No global variables without `our`
- [ ] Use `feature 'signatures'` for modern code (Perl 5.20+)


---


[↑ Back to TOC](#table-of-contents)

---

## 4. Subroutines & Functions

## Parameter Handling

### Modern Approach: Subroutine Signatures

```perl
use strict;
use warnings;
use feature 'signatures';
no warnings 'experimental::signatures';

# ✅ GOOD - Clear, self-documenting signatures
sub calculate_total ($base_price, $tax_rate = 0.10, $discount = 0) {
    my $with_tax = $base_price * (1 + $tax_rate);
    return $with_tax * (1 - $discount);
}

# ✅ GOOD - Handling optional parameters
sub send_email ($to, $subject, $body, $cc = undef, $bcc = undef) {
    # Function body
}

# ✅ GOOD - Slurpy parameters (Perl 5.20+)
sub sum_values (@numbers) {
    my $total = 0;
    $total += $_ for @numbers;
    return $total;
}

# ✅ GOOD - Named hash for options
sub create_user ($username, $email, %options) {
    my $active = $options{active} // 1;
    my $role = $options{role} // 'user';
    # Function body
}
```

### Traditional Approach (Backward Compatible)

If you need to support Perl versions before 5.20:

```perl
# ✅ GOOD - Explicit parameter handling
sub calculate_total {
    my ($base_price, $tax_rate, $discount) = @_;
    
    # Provide defaults
    $tax_rate //= 0.10;
    $discount //= 0;
    
    my $with_tax = $base_price * (1 + $tax_rate);
    return $with_tax * (1 - $discount);
}

# ✅ GOOD - Validate required parameters
sub send_email {
    my ($to, $subject, $body) = @_;
    
    die "send_email: 'to' is required" unless defined $to;
    die "send_email: 'subject' is required" unless defined $subject;
    
    # Function body
}

# ❌ BAD - Silent parameter dropping
sub calculate {
    my $first = $_[0];
    my $second = $_[1];
    # Other parameters silently ignored
}
```

## Explicit Return Statements

Always use explicit `return` statements. They make control flow clear:

```perl
# ✅ GOOD - Explicit early returns
sub validate_user ($user) {
    return 0 unless defined $user;
    return 0 unless $user->{email};
    return 0 unless $user->{password};
    return 1;  # Valid
}

# ✅ GOOD - Explicit return at end
sub calculate_age ($birth_year) {
    my $current_year = (localtime)[5] + 1900;
    my $age = $current_year - $birth_year;
    return $age;
}

# ❌ BAD - Implicit return (confusing)
sub validate_user {
    my ($user) = @_;
    return 0 unless $user;
    $user->{email};  # Last value returned implicitly (unclear)
}

# ❌ BAD - Inconsistent returns
sub process {
    my ($data) = @_;
    if ($data) {
        return 1;
    }
    # Implicitly returns undef if data is false
}
```

## Parameter Validation for Public APIs

Validate inputs at function boundaries, especially for public APIs:

```perl
# ✅ GOOD - Comprehensive validation
sub transfer_funds ($from_account, $to_account, $amount) {
    # Type checking
    die "from_account must be a hash reference"
        unless ref($from_account) eq 'HASH';
    die "to_account must be a hash reference"
        unless ref($to_account) eq 'HASH';
    
    # Value validation
    die "amount must be positive"
        unless defined $amount && $amount > 0;
    
    die "insufficient funds"
        if $from_account->{balance} < $amount;
    
    # Perform operation
    $from_account->{balance} -= $amount;
    $to_account->{balance} += $amount;
    
    return 1;
}

# ✅ GOOD - Using Try::Tiny for cleaner error handling
use Try::Tiny;

sub process_payment ($payment_info) {
    return try {
        validate_payment($payment_info);
        charge_card($payment_info);
        log_transaction($payment_info);
        return { success => 1 };
    }
    catch {
        my $error = $_;
        log_error("Payment failed: $error");
        return { success => 0, error => $error };
    };
}
```

## Single Responsibility Principle

Each function should do one thing well:

```perl
# ❌ BAD - Multiple responsibilities
sub process_and_email_report {
    my ($data) = @_;
    
    # Transforms data
    my @items = sort { $a->{date} cmp $b->{date} } @$data;
    my $total = 0;
    $total += $_->{value} for @items;
    
    # Formats output
    my $report = "Report for " . scalar(@items) . " items\n";
    $report .= "Total: $total\n";
    
    # Sends email
    my $email = Email::Simple->create(
        header => [
            From => 'reports@example.com',
            To => 'admin@example.com',
            Subject => 'Daily Report',
        ],
        body => $report,
    );
    sendmail($email);
    
    return 1;
}

# ✅ GOOD - Separate responsibilities
sub generate_report ($data) {
    my @sorted = sort { $a->{date} cmp $b->{date} } @$data;
    my $total = 0;
    $total += $_->{value} for @sorted;
    
    return {
        items => \@sorted,
        total => $total,
        count => scalar(@sorted),
    };
}

sub format_report ($report) {
    my $text = "Report for $report->{count} items\n";
    $text .= "Total: $report->{total}\n";
    return $text;
}

sub email_report ($formatted_report) {
    my $email = Email::Simple->create(
        header => [
            From => 'reports@example.com',
            To => 'admin@example.com',
            Subject => 'Daily Report',
        ],
        body => $formatted_report,
    );
    sendmail($email);
    return 1;
}

# Usage
my $report = generate_report($data);
my $formatted = format_report($report);
email_report($formatted);
```

## Function Length Guidelines

**Recommendation: Functions should be 50-100 lines maximum**

Longer functions are harder to:
- Understand
- Test
- Modify without breaking
- Reuse

```perl
# ❌ BAD - Function doing too much
sub complex_business_logic {
    # 200+ lines of validation
    # 150+ lines of calculation
    # 100+ lines of formatting
    # 80+ lines of error handling
}

# ✅ GOOD - Decomposed into smaller functions
sub process_order ($order) {
    my $validated = validate_order($order);
    my $calculated = calculate_totals($validated);
    my $formatted = format_for_invoice($calculated);
    my $stored = store_order($formatted);
    return $stored;
}

sub validate_order ($order) {
    # 20-30 lines of validation logic
    return $validated_order;
}

sub calculate_totals ($order) {
    # 15-25 lines of calculation logic
    return $order_with_totals;
}

# etc...
```

## Return Value Conventions

Be consistent about what functions return:

```perl
# ✅ GOOD - Clear return value semantics
# Returns the created resource or undef on failure
sub create_user ($user_data) {
    return undef unless validate_user_data($user_data);
    my $user = insert_into_database($user_data);
    return $user;
}

# Returns boolean for success/failure
sub delete_user ($user_id) {
    return 0 unless user_exists($user_id);
    remove_from_database($user_id);
    return 1;
}

# Returns data or dies on failure
sub fetch_config ($filename) {
    open my $fh, '<', $filename or die "Cannot open config: $!";
    my $config = decode_json(do { local $/; <$fh> });
    close $fh;
    return $config;
}

# ❌ BAD - Inconsistent return patterns
sub maybe_get_data {
    my $data = fetch_data();
    if ($data) {
        return $data;  # Returns data
    } else {
        return 0;      # Returns false (confusing - is 0 data?)
    }
}
```

## Documentation for Functions

Every public function should be documented:

```perl
# ✅ GOOD - POD documentation for public API
=head2 calculate_discount

Calculate discount amount based on purchase total.

=over 4

=item Parameters:
  - $total (number): Purchase total in dollars

=item Returns:
  Number representing discount amount

=item Throws:
  Dies if total is negative

=back

  my $discount = calculate_discount($total);

=cut

sub calculate_discount ($total) {
    die "Total cannot be negative" if $total < 0;
    return 0 if $total < 50;
    return $total * 0.10 if $total < 100;
    return $total * 0.20;
}

# ✅ GOOD - Internal function with comment
# Calculate tax based on state and amount
# (Internal helper - not part of public API)
sub _calculate_tax {
    my ($state, $amount) = @_;
    # Implementation
}
```

## Common Function Patterns

### Guard Clauses (Early Returns)

```perl
# ✅ GOOD - Guard clauses at start
sub process_order ($order) {
    return 0 unless defined $order;
    return 0 unless $order->{items};
    return 0 if $order->{total} <= 0;
    return 0 if $order->{status} eq 'cancelled';
    
    # Main logic only executes if all guards pass
    perform_order_processing($order);
    return 1;
}

# ❌ BAD - Nested conditionals
sub process_order {
    my ($order) = @_;
    if (defined $order) {
        if ($order->{items}) {
            if ($order->{total} > 0) {
                if ($order->{status} ne 'cancelled') {
                    perform_order_processing($order);
                    return 1;
                }
            }
        }
    }
    return 0;
}
```

### Method Chaining (Fluent Interface)

```perl
# ✅ GOOD - Fluent interface
package DataProcessor;

sub new ($class) {
    return bless {}, $class;
}

sub load_file ($self, $filename) {
    $self->{data} = read_file($filename);
    return $self;  # Return self for chaining
}

sub filter ($self, $condition) {
    $self->{data} = [grep { $condition->($_) } @{$self->{data}}];
    return $self;
}

sub transform ($self, $transformer) {
    $self->{data} = [map { $transformer->($_) } @{$self->{data}}];
    return $self;
}

sub get_result ($self) {
    return $self->{data};
}

# Usage - readable chain of operations
my $result = DataProcessor->new()
    ->load_file('input.txt')
    ->filter(sub { $_[0] > 10 })
    ->transform(sub { $_[0] * 2 })
    ->get_result();
```

## Subroutines & Functions Checklist

- [ ] Modern signatures used (Perl 5.20+) or explicit parameter extraction
- [ ] All parameters validated at function start
- [ ] Explicit `return` statements throughout
- [ ] Function does one thing (single responsibility)
- [ ] Function is 50-100 lines maximum
- [ ] Return value semantics are clear and consistent
- [ ] Public functions are documented with POD
- [ ] Early returns (guard clauses) used to reduce nesting
- [ ] No use of `$_` unless in map/grep contexts
- [ ] Error conditions use `die` or `Try::Tiny`


---


[↑ Back to TOC](#table-of-contents)

---

## 5. Variables & Data Structures

## Lexical Scoping with `my`

Always declare variables with `my` to create lexically-scoped variables. This prevents accidental global state and makes code safer:

```perl
# ✅ GOOD - Lexical scope with my
{
    my $local_var = 'value';
    print $local_var;  # OK
}
# print $local_var;  # ERROR - out of scope

# ✅ GOOD - Function scope
sub process {
    my $data = shift;
    my $result = transform($data);
    return $result;
}

# ❌ BAD - Implicit global variable (with strict, would error)
$global_var = 'value';   # Creates package variable

# ❌ BAD - Accidental global if strict isn't enforced
sub calculate {
    $result = 10 + 20;   # Creates package variable!
    return $result;
}
```

### Scope and Lifetime

```perl
# ✅ GOOD - Variables scoped to where they're used
sub get_user_profile ($user_id) {
    my $user = fetch_user($user_id);
    return undef unless $user;
    
    # Create a new scope for user data processing
    {
        my $formatted_user = {
            name => $user->{first} . ' ' . $user->{last},
            email => $user->{email_address},
            verified => $user->{email_verified} ? 'Yes' : 'No',
        };
        return $formatted_user;
    }
    # $formatted_user is out of scope here
}

# ✅ GOOD - Block scope for temporary variables
foreach my $item (@items) {
    my $processed = process_item($item);
    my $validated = validate_result($processed);
    print_result($validated);
}
# $processed and $validated are out of scope after loop
```

## Avoid Package Variables When Possible

Package variables (declared with `our`) should be rare and well-documented:

```perl
# ❌ BAD - Unnecessary package variable
our $counter = 0;
sub increment {
    $counter++;
}

# ✅ GOOD - Use lexical variable instead
my $counter = 0;
sub increment {
    return ++$counter;
}

# ✅ ACCEPTABLE - Package variable for configuration
our $VERSION = '1.0.0';
our $DEBUG = 0;

# ✅ GOOD - When you must use package variables, document them
=head1 PACKAGE VARIABLES

=over 4

=item $debug

Global debug flag. When true, enables verbose logging.

=back

=cut

our $debug = $ENV{DEBUG} // 0;
```

## Reference Usage and Dereferencing

References allow passing complex data structures and creating nested structures:

```perl
# ✅ GOOD - Creating references
my $scalar_ref = \$scalar;
my $array_ref = \@array;
my $hash_ref = \%hash;
my $code_ref = \&subroutine;

# Anonymous references (preferred)
my $array_ref = [1, 2, 3];
my $hash_ref = { name => 'John', age => 30 };
my $code_ref = sub { return $_[0] * 2 };

# ✅ GOOD - Dereferencing scalars
my $value = $$scalar_ref;
$$scalar_ref = 'new value';

# ✅ GOOD - Dereferencing arrays
my $first = $array_ref->[0];
my @all = @$array_ref;
push @$array_ref, 'item';

# ✅ GOOD - Dereferencing hashes
my $name = $hash_ref->{name};
my %all = %$hash_ref;
$hash_ref->{age} = 31;

# ✅ GOOD - Dereferencing code
my $result = $code_ref->(5);

# ❌ BAD - Old-style dereferencing (less readable)
my $first = $$array_ref[0];      # vs. $array_ref->[0]
my $name = $$hash_ref{name};     # vs. $hash_ref->{name}
my $result = &$code_ref(5);      # vs. $code_ref->(5)
```

### Complex Data Structures

```perl
# ✅ GOOD - Well-structured nested data
my $config = {
    database => {
        host => 'localhost',
        port => 5432,
        name => 'myapp',
        credentials => {
            user => 'admin',
            pass => 'secret',
        },
    },
    cache => {
        type => 'redis',
        ttl => 3600,
    },
};

# Access nested structure
my $db_host = $config->{database}->{host};
my $cache_ttl = $config->{cache}->{ttl};

# ✅ GOOD - Array of hashes (common pattern)
my $users = [
    { id => 1, name => 'Alice', email => 'alice@example.com' },
    { id => 2, name => 'Bob',   email => 'bob@example.com'   },
    { id => 3, name => 'Carol', email => 'carol@example.com' },
];

# Iterate through array of hashes
for my $user (@$users) {
    print $user->{name}, "\n";
}

# ✅ GOOD - Hash of arrays
my $teams = {
    engineering => ['Alice', 'Bob', 'Charlie'],
    marketing   => ['Diana', 'Eve'],
    sales       => ['Frank', 'Grace', 'Henry'],
};

# Access
my $eng_team = $teams->{engineering};
print $eng_team->[0];  # Alice
```

## Context Awareness (Scalar vs List Context)

Perl functions often behave differently depending on context:

```perl
# ✅ GOOD - Understanding context
sub get_items {
    my @items = ('a', 'b', 'c');
    return @items;  # Returns list in list context, count in scalar context
}

# List context - @items gets list of values
my @values = get_items();

# Scalar context - $count gets number of items
my $count = get_items();

# ✅ GOOD - Explicit context handling
sub get_data {
    my @records = fetch_records();
    
    # In list context, return the list
    return @records if wantarray;
    
    # In scalar context, return count
    return scalar @records;
}

# ✅ GOOD - Common functions with context awareness
my @lines = split /\n/, $text;    # List context - returns list
my $line_count = split /\n/, $text; # Scalar context - returns count (usually 1)

# ✅ GOOD - localtime context awareness
my ($sec, $min, $hour, $mday, $mon, $year) = localtime;  # List context
my $time_string = localtime;                               # Scalar context
```

### Avoiding Context Bugs

```perl
# ❌ BAD - Context confusion
sub fetch_users {
    my @users = query_database("SELECT * FROM users");
    return @users;  # Returns list in list context
}

my $user_count = fetch_users();  # Scalar context - gets 1 or 0, not count!

# ✅ GOOD - Explicit scalar return when needed
sub fetch_users {
    my @users = query_database("SELECT * FROM users");
    return wantarray ? @users : scalar @users;
}

my $user_count = fetch_users();  # Now gets actual count

# ✅ GOOD - Or use scalar explicitly at call site
my $user_count = scalar fetch_users();
```

## Appropriate Use of `undef` vs Empty Strings

Choose the right "no value" representation:

```perl
# ✅ GOOD - undef for "no value"
my $result = search_database($query);
return undef unless @$result;  # Query found nothing

my $optional_param = undef;    # Parameter not provided
return if !defined $optional_param;

# ✅ GOOD - Empty string for "string with no value"
my $name = '';                 # String that's empty, not missing
my $comment = '';              # Explicitly empty, not undefined

# ✅ GOOD - Zero for "numeric no value"
my $count = 0;                 # Zero items, not undefined
my $balance = 0;               # Zero balance, not undefined

# ❌ BAD - Confusing "no values"
my $data = '';                 # Is this an empty string or no data?
my $status = 0;                # Is this false, zero, or undefined?

# ✅ GOOD - Use defined() and exists() appropriately
if (defined $var) { }          # Check if variable has a value (scalar)
if (exists $hash{key}) { }     # Check if hash key exists
if (@array) { }                # Check if array has elements
if (%hash) { }                 # Check if hash has elements
```

### The `//` (defined-or) Operator

```perl
# ✅ GOOD - Provide defaults with //
my $port = $config{port} // 5432;
my $name = $user->{name} // 'Anonymous';
my $retries = $options{retries} // 3;

# ❌ BAD - Using || with numeric/boolean values
my $count = $data{count} || 0;  # BUG: If count is 0, gets default!
my $enabled = $flags{debug} || 0;  # BUG: If debug is 0, gets default!

# ✅ GOOD - Use || only when you want to treat false values as missing
my $value = get_value() || 'default';  # OK if 0/'' should trigger default
```

## Data Structure Patterns

### Configuration Objects

```perl
# ✅ GOOD - Configuration as hash reference
my $config = {
    app_name => 'MyApp',
    debug => 1,
    database => {
        host => 'localhost',
        port => 5432,
    },
};

# ✅ GOOD - With defaults
my $default_config = {
    debug => 0,
    timeout => 30,
    retries => 3,
};

sub merge_configs ($user_config, $defaults = {}) {
    my %merged = %$defaults;
    @merged{keys %$user_config} = values %$user_config;
    return \%merged;
}

my $final_config = merge_configs($config, $default_config);
```

### Result Objects

```perl
# ✅ GOOD - Explicit result structure
sub process_file ($filename) {
    return {
        success => 0,
        error => "File not found",
    } unless -f $filename;
    
    my $data = read_file($filename);
    return {
        success => 1,
        data => $data,
        size => length $data,
    };
}

# Usage
my $result = process_file('data.txt');
if ($result->{success}) {
    print "Processed " . $result->{size} . " bytes\n";
} else {
    print "Error: " . $result->{error} . "\n";
}
```

### Using Type Checks

```perl
# ✅ GOOD - Verify data structure types
sub process_user ($user) {
    die "user must be a hash reference" unless ref($user) eq 'HASH';
    die "user.id is required" unless $user->{id};
    die "user.name must be a string" unless defined $user->{name};
    
    # Safe to use now
    return format_user($user);
}

# ✅ GOOD - Using isa (Perl 5.32+)
use feature 'isa';

if ($obj isa 'DateTime') {
    print $obj->iso8601;
}

if ($data isa 'ARRAY') {
    my @items = @$data;
}

if ($data isa 'HASH') {
    my %values = %$data;
}
```

## Variables & Data Structures Checklist

- [ ] All variables declared with `my` (lexical scope)
- [ ] No accidental package variables (without `our`)
- [ ] References used appropriately for complex data
- [ ] Dereferencing uses modern arrow syntax (`->`)
- [ ] Data structures well-organized and consistent
- [ ] `undef` used for missing values, not empty strings
- [ ] `defined()` and `exists()` used correctly
- [ ] `//` operator used for default values, not `||`
- [ ] Complex logic uses intermediate named variables
- [ ] Data types documented in function comments
- [ ] No unnecessary global state


---


[↑ Back to TOC](#table-of-contents)

---

## 6. Error Handling

Proper error handling prevents silent failures and makes debugging easier. Use clear error messages and appropriate error reporting mechanisms.

## Die with Context

When an error occurs, `die` should include information about what went wrong:

```perl
# ✅ GOOD - Die with context for file operations
open my $fh, '<', $filename or die "Cannot open '$filename': $!";

# ✅ GOOD - Die with descriptive message
die "Invalid user ID: must be positive" unless $user_id > 0;

# ✅ GOOD - Include variable values in error messages
die "Database connection failed: tried $host:$port, got error: $!" 
    unless $dbh;

# ❌ BAD - Generic error message
open my $fh, '<', $filename or die "Error";

# ❌ BAD - Silent failure
open my $fh, '<', $filename;  # What if this fails?

# ❌ BAD - Missing $! for system calls
open my $fh, '<', $filename or die "Cannot open file";
```

### The `$!` Variable

Always include `$!` when reporting system call errors:

```perl
# ✅ GOOD - Includes system error message
open my $fh, '<', 'data.txt' or die "Cannot open data.txt: $!";
mkdir 'newdir' or die "Cannot create directory: $!";
chdir 'somedir' or die "Cannot change directory: $!";
system('command') or die "Command failed: $!";

# ✅ GOOD - Chain multiple operations with error handling
open my $input, '<', 'input.txt' or die "Cannot open input: $!";
open my $output, '>', 'output.txt' or die "Cannot open output: $!";

# ✅ GOOD - Clear error for specific conditions
my $count = @records or die "No records to process";
my $value = $hash{key} // die "Missing required key: key";
```

## Try::Tiny for Exception Handling

Use `Try::Tiny` for structured exception handling:

```perl
use Try::Tiny;

# ✅ GOOD - Basic try-catch
try {
    my $result = risky_operation($data);
    process_result($result);
}
catch {
    my $error = $_;
    warn "Operation failed: $error";
    return undef;
};

# ✅ GOOD - Multiple catch handlers (custom exception classes)
try {
    validate_and_process($data);
}
catch {
    if (/database error/i) {
        log_database_error($_);
        retry_operation();
    } elsif (/network error/i) {
        log_network_error($_);
        notify_admins($_);
    } else {
        die $_;  # Re-throw unknown errors
    }
};

# ✅ GOOD - Finally block for cleanup
my $file;
try {
    open $file, '<', 'data.txt' or die "Cannot open: $!";
    my $data = do { local $/; <$file> };
    process_data($data);
}
catch {
    warn "Error processing file: $_";
}
finally {
    close $file if $file;  # Always executes
};
```

### Custom Exception Classes

```perl
# ✅ GOOD - Use exception objects for rich error info
package MyApp::Exception;

use strict;
use warnings;

sub new ($class, %args) {
    my $self = {
        message => $args{message} // 'Unknown error',
        code => $args{code} // 500,
        details => $args{details},
    };
    return bless $self, $class;
}

sub throw ($class, %args) {
    die $class->new(%args);
}

# In main code:
use Try::Tiny;

try {
    MyApp::Exception->throw(
        message => 'Invalid user',
        code => 400,
        details => { user_id => $id },
    );
}
catch {
    if (ref($_) eq 'MyApp::Exception') {
        print "Error: " . $_->{message} . "\n";
        log_error($_->{code}, $_->{details});
    } else {
        die $_;  # Re-throw unexpected errors
    }
};
```

## Return Values: Check All System Calls

Always check return values from file operations and system calls:

```perl
# ✅ GOOD - Check open() return
open my $fh, '<', $filename or die "Cannot open: $!";

# ✅ GOOD - Check print() return
print $fh $data or die "Cannot write: $!";

# ✅ GOOD - Check close() return
close $fh or die "Cannot close: $!";

# ✅ GOOD - Check system() return
my $exit_code = system('command', @args);
die "Command failed with code $exit_code" if $exit_code;

# ✅ GOOD - Check chdir() return
chdir $directory or die "Cannot chdir to $directory: $!";

# ❌ BAD - Ignoring return values
open FILE, '<', 'data.txt';     # What if it fails?
print FILE $data;                # What if write fails?
close FILE;                       # What if close fails?
system('command');               # What if command fails?
```

## Guard Clauses and Early Returns

Use guard clauses to handle error conditions early:

```perl
# ✅ GOOD - Guard clauses
sub process_payment ($payment_info) {
    return 0 unless defined $payment_info;
    return 0 unless $payment_info->{amount} > 0;
    return 0 unless validate_card($payment_info);
    return 0 unless $payment_info->{amount} <= MAX_TRANSACTION;
    
    # Only execute main logic if all guards pass
    charge_card($payment_info);
    return 1;
}

# ❌ BAD - Deep nesting
sub process_payment {
    my ($payment_info) = @_;
    if (defined $payment_info) {
        if ($payment_info->{amount} > 0) {
            if (validate_card($payment_info)) {
                if ($payment_info->{amount} <= MAX_TRANSACTION) {
                    charge_card($payment_info);
                    return 1;
                }
            }
        }
    }
    return 0;
}
```

## Error Propagation: When to Die vs Return Undef

Choose the appropriate error reporting mechanism:

```perl
# ✅ GOOD - Die for unexpected/fatal errors
sub read_config_file ($filename) {
    open my $fh, '<', $filename or die "Cannot open config: $!";
    my $config = decode_json(do { local $/; <$fh> });
    close $fh;
    return $config;
}

# ✅ GOOD - Return undef for expected/recoverable errors
sub find_user ($user_id) {
    my $user = $dbh->selectrow_hashref(
        "SELECT * FROM users WHERE id = ?",
        {},
        $user_id
    );
    return undef unless $user;  # User not found (expected)
    return $user;
}

# ✅ GOOD - Return error hash for complex results
sub attempt_operation ($data) {
    return {
        success => 0,
        error => "Invalid data format",
    } unless validate_format($data);
    
    return {
        success => 0,
        error => "Authorization failed",
    } unless authorized_for_operation();
    
    my $result = perform_operation($data);
    return {
        success => 1,
        data => $result,
    };
}

# Usage
my $result = attempt_operation($input);
if ($result->{success}) {
    process($result->{data});
} else {
    warn "Operation failed: " . $result->{error};
}
```

## Logging Errors

Use a proper logging system rather than `print` or `warn`:

```perl
use Log::Log4perl;

my $logger = Log::Log4perl->get_logger('MyApp');

# ✅ GOOD - Use logger instead of warn/print
sub process_record ($record) {
    try {
        validate_record($record);
        $logger->debug("Processing record: $record->{id}");
        
        my $result = transform_record($record);
        $logger->info("Successfully processed record: $record->{id}");
        
        return $result;
    }
    catch {
        my $error = $_;
        $logger->error("Failed to process record: $error");
        $logger->debug("Record data: " . Data::Dumper::Dumper($record));
        
        notify_admin($error) if is_critical_error($error);
        return undef;
    };
}

# ✅ GOOD - Log with context
$logger->warn("High memory usage detected: ${memory}MB (limit: ${max}MB)");
$logger->error("Database connection failed", {
    host => $host,
    port => $port,
    error => $error,
});
```

## Common Error Patterns

### File Operations

```perl
# ✅ GOOD - Complete file operation error handling
use Try::Tiny;

try {
    open my $fh, '<', $filename or die "Cannot open '$filename': $!";
    
    my @lines = <$fh>;
    close $fh or die "Cannot close '$filename': $!";
    
    return \@lines;
}
catch {
    my $error = $_;
    warn "Error reading file: $error";
    return undef;
};
```

### Database Operations

```perl
# ✅ GOOD - Database error handling
my $dbh = DBI->connect(
    "dbi:mysql:database=$db_name;host=$host",
    $user,
    $pass,
    {
        RaiseError => 1,      # Raises exceptions on error
        PrintError => 0,      # Don't print errors to STDERR
        AutoCommit => 1,      # Auto-commit transactions
    }
) or die "Cannot connect to database: $DBI::errstr";

try {
    my $sth = $dbh->prepare("SELECT * FROM users WHERE id = ?");
    $sth->execute($user_id) or die "Query failed: " . $dbh->errstr;
    
    my $user = $sth->fetchrow_hashref;
    return $user;
}
catch {
    my $error = $_;
    $logger->error("Database query failed: $error");
    $dbh->rollback() if $dbh;
    die $error;
};
```

### System Command Execution

```perl
# ✅ GOOD - Safe system call with error handling
use IPC::Run qw(run);

try {
    my @cmd = ('command', 'arg1', 'arg2');
    my ($in, $out, $err);
    
    run \@cmd, \$in, \$out, \$err
        or die "Command failed with exit code: $?";
    
    return $out;
}
catch {
    my $error = $_;
    warn "Command execution failed: $error";
    return undef;
};

# ✅ GOOD - Using system() with proper error checking
my $exit_code = system('command', @args);
if ($exit_code != 0) {
    my $signal = $exit_code & 127;
    my $core_dump = $exit_code & 128;
    my $actual_code = $exit_code >> 8;
    
    die "Command exited with code $actual_code";
}
```

## Error Handling Checklist

- [ ] All file operations check return values or use `or die`
- [ ] System errors include `$!` in message
- [ ] No silent failures (operations always report results)
- [ ] `Try::Tiny` used for complex error handling
- [ ] Guard clauses used to avoid deep nesting
- [ ] Clear distinction between fatal (die) and recoverable (return undef) errors
- [ ] Error messages include context and variable values
- [ ] Logging system used (not `print`/`warn`)
- [ ] Database and external service calls have explicit error handling
- [ ] Custom exception classes used for application-specific errors
- [ ] Finally blocks ensure resource cleanup
- [ ] Error propagation considered at function boundaries


---

Security is not optional. Always validate and sanitize untrusted input. Follow the CERT Perl Secure Coding Standard.


[↑ Back to TOC](#table-of-contents)

---

## 7. Input Validation & Security

## Taint Mode for Scripts Handling Untrusted Input

Enable taint checking when your script processes untrusted data:

```perl
#!/usr/bin/env perl -T
use strict;
use warnings;

# Taint mode is enabled with -T flag
# Any data from outside the script is marked "tainted"
# Tainted data cannot be used in potentially dangerous operations

# ✅ GOOD - With taint mode enabled
my $filename = $ARGV[0];  # TAINTED

# This would die: open() refuses tainted filenames in taint mode
# open my $fh, '<', $filename or die "Cannot open: $!";

# ✅ GOOD - Untaint through explicit validation
if ($filename =~ /^([a-zA-Z0-9._\/-]+)$/) {
    my $safe_filename = $1;  # Untainted
    open my $fh, '<', $safe_filename or die "Cannot open: $!";
}

# ✅ GOOD - Check exit codes (can be tainted)
my $exit = system('command', @args);
$exit = $exit >> 8;  # Untaint by shifting
```

### When to Use Taint Mode

- Scripts that process external input (web requests, file uploads, command-line args)
- CGI scripts and web applications
- System administration scripts
- Any script with security implications

### When Taint Mode Isn't Necessary

- Internal tools processing known data
- Build scripts in controlled environments
- One-off analysis scripts

## Input Validation: Validate All External Data

```perl
# ✅ GOOD - Comprehensive input validation
sub create_user ($username, $email, $age) {
    # Validate username
    die "Username required" unless defined $username && length $username;
    die "Username too long (max 50 chars)" if length $username > 50;
    die "Username invalid" unless $username =~ /^[a-zA-Z0-9_-]+$/;
    
    # Validate email
    die "Email required" unless defined $email && length $email;
    die "Email too long" if length $email > 255;
    die "Invalid email format"
        unless $email =~ /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    
    # Validate age
    die "Age must be a number" unless $age =~ /^\d+$/;
    die "Age must be between 0 and 150" unless $age >= 0 && $age <= 150;
    
    # Safe to use values now
    return create_user_in_db($username, $email, $age);
}

# ❌ BAD - No validation
sub create_user_unsafe {
    my ($username, $email, $age) = @_;
    return create_user_in_db($username, $email, $age);
}
```

## Regular Expression Validation and Untainting

Use regex patterns to validate and untaint data:

```perl
# ✅ GOOD - Validate with regex, extract to untaint
my $user_input = $ARGV[0];  # TAINTED

if ($user_input =~ /^([a-zA-Z0-9._-]+)$/) {
    my $safe_value = $1;  # UNTAINTED
    process_value($safe_value);
} else {
    die "Invalid input format";
}

# ✅ GOOD - More specific validation patterns
# Filename validation
if ($filename =~ m{^([a-zA-Z0-9._-]+\.txt)$}) {
    my $safe_filename = $1;
}

# Number validation
if ($count =~ /^(\d+)$/) {
    my $safe_count = $1;
}

# IPv4 address validation
if ($ip =~ /^(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})$/) {
    my $safe_ip = $1;
    # Still validate ranges
    die "Invalid IP" if grep { $_ > 255 } split /\./, $safe_ip;
}

# ❌ BAD - Too permissive regex
if ($input =~ /^(.*)$/) {  # Matches everything! No untainting
    my $value = $1;
}
```

## PATH Security

Set an explicit safe PATH for system calls:

```perl
# ✅ GOOD - Set safe PATH before system calls
$ENV{PATH} = '/usr/local/bin:/usr/bin:/bin';
delete @ENV{qw(IFS CDPATH ENV BASH_ENV)};

# Now safe to execute commands
system('ls', $directory);

# ✅ GOOD - Or use indirect form (safer)
use IPC::Run qw(run);

my @cmd = ('/usr/bin/ls', $directory);
run \@cmd or die "Command failed: $?";

# ❌ BAD - Using shell with tainted data
my $directory = $ARGV[0];
system("ls $directory");  # SHELL INJECTION VULNERABILITY!
```

## SQL Injection Prevention

Always use parameterized queries with placeholders:

```perl
use DBI;

my $dbh = DBI->connect($dsn, $user, $pass);

# ✅ GOOD - Parameterized query (safe)
my $sth = $dbh->prepare("SELECT * FROM users WHERE email = ? AND active = ?");
$sth->execute($user_email, 1);

# ✅ GOOD - Named placeholders
my $sth = $dbh->prepare(
    "SELECT * FROM users WHERE email = :email AND status = :status"
);
$sth->execute({ email => $user_email, status => 'active' });

# ❌ DANGEROUS - String interpolation (SQL injection)
my $query = "SELECT * FROM users WHERE email = '$user_email'";
my $sth = $dbh->prepare($query);

# Even with escaping, use placeholders instead:
# ❌ BAD - Escape functions are error-prone
my $safe_email = $dbh->quote($user_email);
my $query = "SELECT * FROM users WHERE email = $safe_email";
```

### ORMs and SQL Builders

Use ORMs or SQL builder libraries for complex queries:

```perl
use DBIx::Class;
use SQL::Abstract;

# ✅ GOOD - Using DBIx::Class (automatic parameterization)
my $user = $schema->resultset('User')->find({
    email => $user_email,
    active => 1,
});

# ✅ GOOD - Using SQL::Abstract
my ($stmt, @bind) = $sql->select('users', '*', {
    email => $user_email,
    active => 1,
});

# Then use with placeholders
my $sth = $dbh->prepare($stmt);
$sth->execute(@bind);
```

## Command Injection Prevention

Never pass untrusted data directly to shell:

```perl
# ✅ GOOD - Use list form (avoids shell parsing)
system('command', $arg1, $arg2);

# ✅ GOOD - Use IPC::Run for more control
use IPC::Run qw(run);
my @cmd = ('command', 'arg1', 'arg2');
run \@cmd or die "Failed: $?";

# ✅ GOOD - Validate and untaint before using
if ($filename =~ /^([a-zA-Z0-9._-]+)$/) {
    my $safe = $1;
    system('cat', $safe);
}

# ❌ DANGEROUS - Shell interpretation
my $filename = $ARGV[0];
system("cat $filename");      # Shell injection!
system("cat '$filename'");    # Shell injection! (quotes don't help)

# ❌ DANGEROUS - Using backticks
my $output = `cat $filename`;  # Shell injection!
```

## File Operations Security

Validate filenames and check for path traversal:

```perl
# ✅ GOOD - Validate and untaint filename
sub read_user_file ($filename) {
    # Only allow safe characters
    die "Invalid filename" unless $filename =~ /^([a-zA-Z0-9._-]+)$/;
    my $safe_name = $1;
    
    # Construct full path in controlled directory
    my $base_dir = '/var/app/data';
    my $full_path = File::Spec->catfile($base_dir, $safe_name);
    
    # Verify it's still in base directory (no ../ traversal)
    my $real_path = Cwd::realpath($full_path);
    die "Path traversal attempt" unless $real_path =~ /^\Q$base_dir\E/;
    
    # Now safe to open
    open my $fh, '<', $real_path or die "Cannot open: $!";
    my $content = do { local $/; <$fh> };
    close $fh;
    return $content;
}

# ✅ GOOD - Using File::Temp for temporary files
use File::Temp qw(tempfile);

my ($fh, $filename) = tempfile(DIR => '/var/app/tmp');
print $fh $data;
close $fh;

# ❌ BAD - Predictable temp file names
my $temp = "/tmp/data_$$.txt";  # Can be guessed/exploited
```

## HTTP/Web Security

When handling web input:

```perl
use CGI;
use URI::URL;

my $cgi = CGI->new;

# ✅ GOOD - Validate query parameters
my $user_id = $cgi->param('user_id');
die "Invalid user_id" unless defined $user_id && $user_id =~ /^(\d+)$/;
my $safe_id = $1;

# ✅ GOOD - Escape HTML output
my $user_name = $cgi->param('name');
my $escaped = CGI::escapeHTML($user_name);
print "<p>Hello, $escaped</p>";

# ✅ GOOD - Validate URLs
my $redirect = $cgi->param('redirect_to');
if ($redirect =~ m{^(/[a-zA-Z0-9/_.-]+)$}) {  # Only allow relative URLs
    my $safe_url = $1;
    print $cgi->redirect($safe_url);
}

# ❌ BAD - Output without escaping (XSS)
my $name = $cgi->param('name');
print "<p>Hello, $name</p>";  # If name = "<script>alert(1)</script>"

# ❌ BAD - Unvalidated redirect (Open Redirect)
print $cgi->redirect($cgi->param('goto'));
```

## Cryptographic Operations

Use established libraries for cryptographic operations:

```perl
use Crypt::Bcrypt;
use Digest::SHA qw(sha256_hex);

# ✅ GOOD - Use bcrypt for password hashing
my $password = 'user_password';
my $hashed = Crypt::Bcrypt::bcrypt($password, Crypt::Bcrypt::en_base64(rand() x 16));

# ❌ BAD - Don't create your own crypto
my $hashed = sha256_hex($password . $salt);  # Vulnerable!

# ✅ GOOD - For non-password hashing (e.g., checksums)
use Digest::SHA qw(sha256_hex);
my $checksum = sha256_hex($data);

# ❌ BAD - MD5 for anything security-related
use Digest::MD5 qw(md5_hex);
my $hash = md5_hex($password);  # Cryptographically broken!
```

## Environment Variables

Be cautious with environment variables:

```perl
# ✅ GOOD - Validate environment variables
my $db_host = $ENV{DB_HOST} // 'localhost';
die "Invalid DB_HOST" unless $db_host =~ /^([a-zA-Z0-9.-]+)$/;

my $db_port = $ENV{DB_PORT} // 5432;
die "Invalid DB_PORT" unless $db_port =~ /^(\d+)$/;

# ✅ GOOD - Use safe defaults
my $debug = $ENV{DEBUG} ? 1 : 0;

# ❌ BAD - Trust environment variables directly
my $db_host = $ENV{DB_HOST};      # Could be anything
my $shell_cmd = "ssh $ENV{HOST}"; # Shell injection!
```

## Input Validation Checklist

- [ ] All external input is validated before use
- [ ] Taint mode (-T) used when processing untrusted data
- [ ] Regex patterns are specific and not overly permissive
- [ ] Data is untainted through explicit extraction
- [ ] SQL queries use parameterized statements with placeholders
- [ ] System commands avoid shell interpretation (list form)
- [ ] Filenames are validated and path traversal is prevented
- [ ] HTML output is properly escaped
- [ ] URLs are validated and restricted appropriately
- [ ] Passwords use bcrypt, not MD5 or custom crypto
- [ ] Environment variables are validated
- [ ] No use of `eval()` with untrusted data
- [ ] File permissions are checked (e.g., `stat()`)


---

Testing is crucial for code reliability. Use Test::More as the standard testing framework.


[↑ Back to TOC](#table-of-contents)

---

## 8. Testing Best Practices

## Test Organization

### Standard Directory Structure

```
project/
├── lib/
│   └── MyApp/
│       ├── User.pm
│       ├── Database.pm
│       └── Utils.pm
├── t/
│   ├── 00-load.t              # Basic module loading
│   ├── 01-basic.t             # Basic functionality
│   ├── 10-user.t              # User module tests
│   ├── 20-database.t          # Database module tests
│   ├── 30-integration.t       # Integration tests
│   └── fixtures/              # Test data files
│       ├── users.json
│       └── config.yml
├── t/lib/                      # Test utilities
│   └── TestHelper.pm
├── Makefile.PL
└── README.md
```

### Test File Naming

- `00-load.t` - Load all modules without errors
- `01-*.t` - Basic sanity tests
- `10-module.t` - Tests for specific modules
- `90-*.t` - Performance or stress tests

## Test::More Standard Functions

### Basic Assertions

```perl
use Test::More tests => 12;  # Declare test count upfront

# String equality
is($result, $expected, "Description of test");
isnt($result, $not_expected, "Should not be equal");

# Numeric comparison
ok($value, "Value is true");
ok($value > 0, "Value is positive");

# Pattern matching
like($string, qr/pattern/, "String matches pattern");
unlike($string, qr/pattern/, "String doesn't match pattern");

# Data structure comparison
is_deeply($got, $expected, "Data structures match");

# Existence checks
ok(defined $value, "Value is defined");
ok(exists $hash{key}, "Hash key exists");
ok(@array, "Array is not empty");

# Undef checking
is($value, undef, "Value is undefined");
```

### Module Loading

```perl
use Test::More tests => 3;

use_ok('MyApp::User');           # Can we use the module?
can_ok('MyApp::User', 'new');    # Does it have the method?
can_ok('MyApp::User', ['new', 'get_name', 'set_email']);
```

### Test Planning

```perl
# ✅ GOOD - Fixed test count
use Test::More tests => 5;
# ... 5 tests must follow ...

# ✅ GOOD - Unknown test count (use done_testing)
use Test::More;
# ... tests ...
done_testing();  # Counts and reports total tests

# ✅ GOOD - Skip tests conditionally
use Test::More;
my $skip_count = 2;
SKIP: {
    skip "Database not available", $skip_count unless $db_available;
    ok(create_user('test'), "Can create user");
    ok(delete_user('test'), "Can delete user");
}
done_testing();
```

## Descriptive Test Labels

Every test needs a clear, descriptive label:

```perl
# ✅ GOOD - Clear descriptions
ok($user->is_active, "New users are active by default");
is($user->role, 'member', "New users have member role");
like($email, qr/@/, "Email contains @");

# ❌ BAD - No or vague descriptions
ok($user->is_active);
is($user->role, 'member');
like($email, qr/@/);

# ❌ BAD - Redundant descriptions (just narrate what code does)
ok($user->is_active, "Call is_active method");
is($user->role, 'member', "Assign member role");
```

## Subtests for Organization

Group related tests in subtests:

```perl
use Test::More tests => 1;

# ✅ GOOD - Organize tests in subtests
subtest "User creation" => sub {
    my $user = MyApp::User->new(name => 'Alice', email => 'alice@example.com');
    ok($user, "User object created");
    is($user->name, 'Alice', "Name set correctly");
    is($user->email, 'alice@example.com', "Email set correctly");
};

subtest "User validation" => sub {
    my $user = MyApp::User->new();
    my @errors = $user->validate();
    is(scalar @errors, 2, "Two validation errors");
    like($errors[0], qr/name/, "First error about name");
    like($errors[1], qr/email/, "Second error about email");
};

subtest "User updates" => sub {
    my $user = MyApp::User->new(name => 'Bob', email => 'bob@example.com');
    $user->set_email('bob2@example.com');
    is($user->email, 'bob2@example.com', "Email updated");
};
```

## Mocking External Dependencies

Use Test::MockModule to mock external dependencies:

```perl
use Test::More tests => 3;
use Test::MockModule;

# ✅ GOOD - Mock external service
my $mock_db = Test::MockModule->new('MyApp::Database');
$mock_db->mock('query', sub {
    return [
        { id => 1, name => 'Alice' },
        { id => 2, name => 'Bob' },
    ];
});

# Now test code that uses the database
my $users = MyApp::UserService::fetch_all_users();
is(scalar @$users, 2, "Fetched 2 users");
is($users->[0]->{name}, 'Alice', "First user is Alice");

# ✅ GOOD - Mock with side effects
my $mock_email = Test::MockModule->new('MyApp::EmailService');
my @sent_emails;
$mock_email->mock('send', sub {
    my ($to, $subject, $body) = @_;
    push @sent_emails, { to => $to, subject => $subject, body => $body };
    return 1;
});

MyApp::UserService::send_welcome_email('alice@example.com');
is(scalar @sent_emails, 1, "Email sent");
is($sent_emails[0]->{to}, 'alice@example.com', "Email to correct address");
```

### Test Fixtures and Setup

```perl
# ✅ GOOD - Setup and teardown in fixtures
package TestHelper;

use strict;
use warnings;

sub setup_test_db {
    my $dbh = DBI->connect('dbi:SQLite::memory:');
    $dbh->do(q{
        CREATE TABLE users (
            id INTEGER PRIMARY KEY,
            name TEXT,
            email TEXT
        )
    });
    return $dbh;
}

sub populate_test_data {
    my ($dbh) = @_;
    $dbh->do("INSERT INTO users (name, email) VALUES (?, ?)", {}, 'Alice', 'alice@example.com');
    $dbh->do("INSERT INTO users (name, email) VALUES (?, ?)", {}, 'Bob', 'bob@example.com');
}

1;

# In test file:
use Test::More tests => 2;
use TestHelper;

my $dbh = TestHelper::setup_test_db();
TestHelper::populate_test_data($dbh);

my @users = @{$dbh->selectall_arrayref("SELECT * FROM users", { Slice => {} })};
is(scalar @users, 2, "Test data populated");
is($users[0]->{name}, 'Alice', "First user is Alice");
```

## Test Coverage

Aim for 80%+ test coverage:

```bash
# Run tests with coverage reporting
perl -MDevel::Cover=-ignore_dirs,t t/*.t
cover -report html  # Generates HTML coverage report
```

### Example .coveragerc Configuration

```ini
# Ignore certain patterns
--ignore_dirs t lib/My
--ignore_pattern ^t/
```

## Common Test Patterns

### Testing Functions

```perl
use Test::More tests => 4;

sub add ($a, $b) { return $a + $b; }

is(add(2, 3), 5, "2 + 3 = 5");
is(add(-1, 1), 0, "-1 + 1 = 0");
is(add(0, 0), 0, "0 + 0 = 0");
is(add(100, 200), 300, "100 + 200 = 300");
```

### Testing Error Conditions

```perl
use Test::More tests => 3;
use Try::Tiny;

sub divide ($a, $b) {
    die "Cannot divide by zero" if $b == 0;
    return $a / $b;
}

is(divide(10, 2), 5, "Normal division works");

my $error;
try {
    divide(10, 0);
}
catch {
    $error = $_;
};
like($error, qr/divide by zero/, "Throws error on division by zero");
ok($error, "Error was caught");
```

### Testing Object Methods

```perl
use Test::More tests => 5;

my $user = MyApp::User->new(name => 'Alice', email => 'alice@example.com');

isa_ok($user, 'MyApp::User', "Created correct object type");
can_ok($user, 'get_name');
is($user->get_name, 'Alice', "get_name returns correct value");

$user->set_email('alice2@example.com');
is($user->get_email, 'alice2@example.com', "Email updated via setter");

is_deeply(
    $user->to_hash,
    { name => 'Alice', email => 'alice2@example.com' },
    "to_hash returns correct structure"
);
```

### Testing Array/List Functions

```perl
use Test::More tests => 4;

sub filter_even {
    my (@numbers) = @_;
    return grep { $_ % 2 == 0 } @numbers;
}

my @result = filter_even(1, 2, 3, 4, 5, 6);
is_deeply(\@result, [2, 4, 6], "Correctly filters even numbers");

@result = filter_even();
is_deeply(\@result, [], "Empty input returns empty array");

@result = filter_even(1, 3, 5);
is_deeply(\@result, [], "No even numbers returns empty array");

@result = filter_even(2, 4, 6);
is_deeply(\@result, [2, 4, 6], "All even numbers returns all");
```

## Complete Test File Example

```perl
#!/usr/bin/env perl
use strict;
use warnings;

use Test::More;
use Test::MockModule;
use Try::Tiny;

use MyApp::UserService;

# Test setup
subtest "Module loads" => sub {
    use_ok('MyApp::UserService');
    can_ok('MyApp::UserService', 'create_user');
    can_ok('MyApp::UserService', 'find_user');
};

# Mock database
my $mock_db = Test::MockModule->new('MyApp::Database');
my %users;
$mock_db->mock('insert', sub {
    my ($table, $data) = @_;
    my $id = scalar(keys %users) + 1;
    $users{$id} = $data;
    return $id;
});
$mock_db->mock('find_by_id', sub {
    my ($table, $id) = @_;
    return $users{$id};
});

# Test user creation
subtest "User creation" => sub {
    my $user_id = MyApp::UserService::create_user('alice@example.com');
    ok($user_id, "User created");
    ok($users{$user_id}, "User saved to database");
};

# Test validation
subtest "User validation" => sub {
    my $error;
    try {
        MyApp::UserService::create_user('');
    }
    catch {
        $error = $_;
    };
    like($error, qr/email required/, "Validates empty email");
};

done_testing();
```

## Testing Best Practices Checklist

- [ ] Tests organized in `t/` directory
- [ ] Test files named descriptively (00-, 10-, etc.)
- [ ] All tests have clear descriptive labels
- [ ] Test count declared upfront or `done_testing()` used
- [ ] Related tests grouped in subtests
- [ ] External dependencies mocked with Test::MockModule
- [ ] Test fixtures used for common setup
- [ ] Error conditions tested with Try::Tiny
- [ ] Both happy path and edge cases tested
- [ ] Test coverage monitored (target 80%+)
- [ ] No test interdependencies (each test runs independently)
- [ ] Tests run in seconds (fast feedback)
- [ ] Tests are deterministic (no random failures)


---

Good documentation makes code usable and maintainable. Use POD (Plain Old Documentation) for modules and write clear comments for complex logic.


[↑ Back to TOC](#table-of-contents)

---

## 9. Documentation

## POD (Plain Old Documentation) for Modules

POD is Perl's built-in documentation format. It's embedded directly in code and extracted by perldoc.

### Module Documentation Structure

```perl
package MyApp::User;

use strict;
use warnings;

=head1 NAME

MyApp::User - User object management

=head1 SYNOPSIS

    use MyApp::User;
    
    my $user = MyApp::User->new(
        id    => 1,
        name  => 'Alice',
        email => 'alice@example.com',
    );
    
    print $user->name;      # Alice
    $user->set_email('alice2@example.com');

=head1 DESCRIPTION

MyApp::User provides an object-oriented interface to user data. It handles
user creation, validation, and common operations.

Users can be created with required fields (id, name, email) and optional
fields like role, status, etc.

=head1 ATTRIBUTES

=head2 id

Unique identifier for the user. Assigned by database.

=head2 name

User's full name. Required at creation time.

=head2 email

User's email address. Must be unique in the system.

=head1 METHODS

=head2 new(%args)

Constructor. Creates a new user object.

Parameters:

=over 4

=item id (required)

Unique identifier

=item name (required)

User's full name

=item email (required)

User's email address

=item role (optional, default: 'user')

User's role in the system

=back

Returns the new user object or undef if validation fails.

Throws an exception if required parameters are missing.

    my $user = MyApp::User->new(
        id    => 1,
        name  => 'Alice',
        email => 'alice@example.com',
    );

=head2 get_name()

Returns the user's name.

    my $name = $user->get_name();

=head2 set_email($new_email)

Updates the user's email address.

Validates the new email format before updating.

Parameters:

=over 4

=item $new_email (string, required)

New email address

=back

Throws exception if email format is invalid.

    $user->set_email('newemail@example.com');

=head2 is_active()

Returns 1 if user is active, 0 otherwise.

    if ($user->is_active()) {
        print "User is active\n";
    }

=head1 EXAMPLES

=head2 Creating and modifying users

    use MyApp::User;
    
    my $user = MyApp::User->new(
        id    => 123,
        name  => 'Bob',
        email => 'bob@example.com',
    );
    
    $user->set_email('bob2@example.com');
    print $user->get_name();

=head2 Error handling

    use MyApp::User;
    use Try::Tiny;
    
    try {
        my $user = MyApp::User->new(
            id    => 456,
            name  => 'Carol',
            email => 'invalid-email',  # Bad format
        );
    }
    catch {
        warn "User creation failed: $_";
    };

=head1 DIAGNOSTICS

=head2 "Missing required parameter: name"

The name parameter is required but was not provided.

=head2 "Invalid email format"

The email address provided doesn't match expected format.

=head1 SEE ALSO

L<MyApp::Database> - Database operations

L<MyApp::UserService> - Higher-level user management

L<MyApp::Validator> - Input validation utilities

=head1 AUTHOR

Your Name <your.email@example.com>

=head1 LICENSE AND COPYRIGHT

Copyright (C) 2026 Your Company

This module is free software; you can redistribute it and/or modify it
under the same terms as Perl itself.

=cut

# Code starts here
our $VERSION = '1.0.0';

sub new {
    # implementation
}

# ... rest of module ...

1;  # Module must end with true value
```

### Viewing POD Documentation

```bash
# View POD in terminal
perldoc MyApp::User

# View specific sections
perldoc -s METHODS MyApp::User

# Generate HTML
pod2html MyApp/User.pm > user.html

# Generate man page
pod2man MyApp/User.pm > user.1
```

## Function and Subroutine Documentation

Document all public functions in POD:

```perl
=head2 calculate_discount($total, $customer_type)

Calculate discount amount based on purchase total and customer type.

Parameters:

=over 4

=item $total (number, required)

Purchase total in dollars. Must be positive.

=item $customer_type (string, optional)

Type of customer: 'regular', 'premium', 'vip'. Default: 'regular'

=back

Returns:

Discount amount as a number. Returns 0 if no discount applies.

Throws:

Dies if $total is negative or not a number.

Examples:

    my $discount = calculate_discount(100, 'premium');      # $10
    my $discount = calculate_discount(50, 'regular');       # $0
    my $discount = calculate_discount(500, 'vip');          # $100

=cut

sub calculate_discount ($total, $customer_type = 'regular') {
    die "Total must be positive" unless $total > 0;
    
    # discount logic...
}
```

## Comments: Explain Why, Not What

Write comments that explain the reasoning behind code, not just narrate what it does:

```perl
# ✅ GOOD - Explains why (the intent/decision)
# Cache regex patterns for performance optimization since we compile them
# on every email validation call (profile showed 40% of time in regex compilation)
my $email_pattern = qr/^[\w\.-]+@[\w\.-]+\.\w{2,}$/;

sub validate_email ($email) {
    return $email =~ $email_pattern;
}

# ✅ GOOD - Documents important edge case
# Note: We use // instead of || here because score can be 0 (which is valid)
my $final_score = $calculated_score // DEFAULT_SCORE;

# ✅ GOOD - Explains non-obvious business logic
# Premium customers get 20% discount, but during sales we cap all discounts
# at 15% to preserve margin. See ticket #1234 for discussion.
my $discount = min($base_discount, 0.15) if $is_during_sale;

# ❌ BAD - Just narrates what the code does
# Calculate discount
my $discount = $amount * 0.10;

# ❌ BAD - Obvious comment adds no value
# Check if email is valid
if ($email =~ /@/) {
    # ...
}

# ❌ BAD - Comments that should be in code or tests
# This function has a bug where it returns wrong value for negative numbers
# TODO: Fix this before production
```

### When to Use Comments

- Explain non-obvious business logic or requirements
- Document performance optimizations and why they're necessary
- Link to external documentation or tickets
- Warn about gotchas or edge cases
- Explain temporary workarounds
- Document assumptions

### When NOT to Use Comments

- When the code itself is clear (variable names, function names explain intent)
- To restate what obvious code does
- When a better refactoring would eliminate the need for comments

## Version Numbers

Use semantic versioning for modules:

```perl
package MyApp::UserService;

our $VERSION = '2.1.3';

# Format: MAJOR.MINOR.PATCH
# MAJOR: Incompatible API changes
# MINOR: New functionality (backward compatible)
# PATCH: Bug fixes

# Before version 1.0.0, use 0.VERSION
# 0.1.0 - alpha release
# 0.5.0 - beta release
# 1.0.0 - stable release
```

## LICENSE and COPYRIGHT

Include license information in all modules:

```perl
=head1 LICENSE AND COPYRIGHT

Copyright (C) 2026 Your Company, Inc.

This module is free software; you can redistribute it and/or modify it
under the same terms as Perl itself (either the Artistic License or the
GPL, at your option).

For details, see http://dev.perl.org/licenses/

=cut
```

## README and Project Documentation

Create comprehensive README.md:

```markdown
# MyApp

Short description of what this application/module does.

## Installation

    cpan MyApp

Or from source:

    perl Makefile.PL
    make
    make test
    make install

## Requirements

- Perl 5.20+
- List of key dependencies:
  - DBI 1.600+
  - DBD::mysql 4.000+
  - Try::Tiny
  - Log::Log4perl

## Quick Start

    use MyApp;
    my $app = MyApp->new(config_file => 'app.conf');
    $app->run();

## Configuration

See CONFIGURATION.md for detailed configuration options.

## Testing

    perl Makefile.PL
    make test

Run tests with coverage:

    prove -v t/

## Documentation

Full documentation available via:

    perldoc MyApp

View specific modules:

    perldoc MyApp::Module

Generate HTML docs:

    pod2html lib/MyApp.pm > docs.html

## Examples

See the `examples/` directory for working examples.

## License

This software is released under the Perl Artistic License. See LICENSE for details.

## Contributing

Please submit bug reports and pull requests to:
https://github.com/yourname/myapp

## Author

Your Name <email@example.com>
```

## Change Log

Maintain a CHANGELOG.md:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.1.3] - 2026-03-02

### Added
- New filtering options for user searches
- Database connection pooling

### Fixed
- Bug in email validation regex (issue #245)
- Memory leak in long-running processes

### Changed
- Improved error messages for validation failures
- Updated dependencies

## [2.1.2] - 2026-02-15

### Fixed
- Database timeout issues

## [2.1.1] - 2026-02-01

### Added
- Initial feature support

...
```

## API Documentation

For public APIs, include detailed documentation:

```perl
=head1 PUBLIC API

The following methods are guaranteed stable and can be relied upon in
production code:

- new(%args)
- find_user($id)
- create_user(%data)
- delete_user($id)

Methods prefixed with underscore (_) are private and not part of the
public API.

=head1 PRIVATE METHODS

The following methods are for internal use only and may change without
notice:

=head2 _validate_email($email)

Internal validation helper.

=head2 _log_action($action)

Internal logging helper.

=cut
```

## Inline Code Examples

Include runnable examples in documentation:

```perl
=head1 EXAMPLES

=head2 Basic Usage

    use MyApp::User;
    
    my $user = MyApp::User->new(
        name  => 'Alice',
        email => 'alice@example.com',
    );
    
    print $user->name;  # Alice

=head2 Error Handling

    use MyApp::User;
    use Try::Tiny;
    
    try {
        my $user = MyApp::User->new(
            name  => 'Bob',
            email => '',  # Invalid
        );
    }
    catch {
        warn "Error: $_";
    };

=head2 Working with Collections

    use MyApp::UserService;
    
    my @active_users = MyApp::UserService->get_active_users();
    
    for my $user (@active_users) {
        print $user->name, "\n";
    }

=cut
```

## Documentation Checklist

- [ ] All modules have NAME, SYNOPSIS, DESCRIPTION sections
- [ ] All public methods/functions documented in POD
- [ ] Parameters and return values documented
- [ ] Usage examples provided
- [ ] Error conditions documented
- [ ] Complex logic has explanatory comments
- [ ] Version numbers use semantic versioning
- [ ] LICENSE and COPYRIGHT included
- [ ] README.md exists with quick start guide
- [ ] CHANGELOG.md maintained
- [ ] API stability documented (public vs private)
- [ ] External references and links work
- [ ] No typos or grammar errors
- [ ] Documentation stays current with code


---

Building reusable, well-structured modules is essential for maintainable Perl projects.


[↑ Back to TOC](#table-of-contents)

---

## 10. Module Development

## Package Declarations

Always use explicit package names at the top of modules:

```perl
package MyApp::User;

use strict;
use warnings;

our $VERSION = '1.0.0';

# ... rest of module ...

1;  # Module must return true value
```

### Best Practices

```perl
# ✅ GOOD - Clear package structure
package MyApp::Database::Query;    # Purpose is clear from name

use strict;
use warnings;

# ✅ GOOD - Explicit version
our $VERSION = '2.3.1';

# ✅ GOOD - Module ends with true value (required)
1;

# ❌ BAD - Package name doesn't indicate purpose
package Utils;

# ❌ BAD - Missing version
# (Version should always be present)

# ❌ BAD - Missing return value at end
# (Module will fail to load)
```

## Module Version Numbers

Use semantic versioning and declare explicitly:

```perl
package MyApp;

our $VERSION = '1.0.0';

# Development versions before 1.0.0
our $VERSION = '0.1.0';  # Alpha
our $VERSION = '0.5.0';  # Beta
our $VERSION = '0.9.0';  # Release candidate

# Access version in code
sub get_version {
    return $VERSION;
}

# Users can query version
use MyApp 1.0;  # Requires version 1.0 or later
```

## Exporting Symbols

Use Exporter or Exporter::Tiny for clean public APIs:

### Using Exporter

```perl
package MyApp::Utils;

use strict;
use warnings;
use base 'Exporter';

our @EXPORT    = qw(format_date parse_date);      # Always exported
our @EXPORT_OK = qw(validate_date);               # Export if requested
our %EXPORT_TAGS = (
    dates => [qw(format_date parse_date validate_date)],
);

our $VERSION = '1.0.0';

# Exported functions
sub format_date ($date) {
    return $date->strftime('%Y-%m-%d');
}

sub parse_date ($date_string) {
    return DateTime::Format::ISO8601->parse_datetime($date_string);
}

# Optional export
sub validate_date ($date) {
    return defined $date && $date->isa('DateTime');
}

1;

# Usage in other modules:
use MyApp::Utils;                    # Gets format_date, parse_date
use MyApp::Utils qw(validate_date);  # Explicitly request optional
use MyApp::Utils ':dates';           # All date functions
```

### Using Exporter::Tiny (Recommended)

```perl
package MyApp::Utils;

use strict;
use warnings;
use Exporter::Tiny;

our @EXPORT    = qw(format_date parse_date);
our @EXPORT_OK = qw(validate_date);

our $VERSION = '1.0.0';

# Implementation...

1;
```

## Object-Oriented Perl

### Choosing an OO Framework

**Moo**: Lightweight, recommended for most new code

```perl
package MyApp::User;

use strict;
use warnings;
use Moo;

has 'id'    => (is => 'ro', required => 1);
has 'name'  => (is => 'rw', required => 1);
has 'email' => (is => 'rw', required => 1);

sub greet ($self) {
    return "Hello, " . $self->name;
}

1;

# Usage:
use MyApp::User;

my $user = MyApp::User->new(
    id    => 1,
    name  => 'Alice',
    email => 'alice@example.com',
);

print $user->greet();  # Hello, Alice
$user->name('Alicia'); # Update name
```

**Moose**: More features if you need them

```perl
package MyApp::User;

use strict;
use warnings;
use Moose;

has 'id' => (
    is       => 'ro',
    isa      => 'Int',
    required => 1,
);

has 'name' => (
    is       => 'rw',
    isa      => 'Str',
    required => 1,
    trigger  => sub {
        my ($self, $new_value) = @_;
        # Code runs whenever name is changed
    },
);

has 'email' => (
    is      => 'rw',
    isa     => 'Email::Valid',
    default => sub { '' },
);

1;
```

**Bare Perl**: For simple cases or when dependencies matter

```perl
package MyApp::User;

use strict;
use warnings;

sub new ($class, %args) {
    my $self = {
        id    => $args{id},
        name  => $args{name},
        email => $args{email},
    };
    return bless $self, $class;
}

sub name ($self, $new_name = undef) {
    if (defined $new_name) {
        $self->{name} = $new_name;
    }
    return $self->{name};
}

1;
```

## Constructor Best Practices

### Named Parameters

```perl
# ✅ GOOD - Named parameters (clearer)
my $user = MyApp::User->new(
    id    => 1,
    name  => 'Alice',
    email => 'alice@example.com',
);

# ❌ BAD - Positional parameters (easy to get wrong)
my $user = MyApp::User->new(1, 'Alice', 'alice@example.com');

# ❌ BAD - Wrong order breaks code silently
my $user = MyApp::User->new('Alice', 1, 'alice@example.com');
```

### Constructor Implementation

```perl
use Moo;

# ✅ GOOD - Moo handles construction
has 'name' => (is => 'rw', required => 1);

# Or implement manually:
# ✅ GOOD - Explicit validation in constructor
sub new ($class, %args) {
    my $name = $args{name} or die "name is required";
    die "name must be a string" unless defined $name && length $name;
    
    my $self = {
        name => $name,
        created_at => time(),
    };
    
    return bless $self, $class;
}

# ✅ GOOD - With defaults
sub new ($class, %args) {
    my $self = {
        name => $args{name} // 'Unknown',
        active => $args{active} // 1,
        role => $args{role} // 'user',
    };
    return bless $self, $class;
}
```

## Accessor Methods

Clear, consistent accessors improve usability:

```perl
use Moo;

# ✅ GOOD - Moo handles accessors automatically
has 'name' => (is => 'rw');         # Read-write accessor
has 'id'   => (is => 'ro');         # Read-only
has 'created_at' => (
    is      => 'ro',
    default => sub { time() },      # Lazy default
);

# Or implement manually:
sub name ($self, $new_name = undef) {
    if (defined $new_name) {
        $self->{name} = $new_name;
        return $new_name;
    }
    return $self->{name};
}

sub get_name ($self) {
    return $self->{name};
}

sub set_name ($self, $new_name) {
    die "Name cannot be empty" unless defined $new_name && length $new_name;
    $self->{name} = $new_name;
    return $self;  # For chaining
}
```

## Inheritance and Composition

Prefer composition over inheritance:

```perl
# ❌ AVOID - Deep inheritance hierarchies
# Base -> User -> ActiveUser -> PremiumUser -> AdminUser

# ✅ GOOD - Composition with roles
use Moo;
use Moo::Role;

role 'HasPermissions' => sub {
    has 'permissions' => (is => 'ro', default => sub { [] });
    
    method 'can_do' => sub ($self, $action) {
        return grep { $_ eq $action } @{$self->permissions};
    };
};

role 'Activatable' => sub {
    has 'active' => (is => 'rw', default => 1);
    
    method 'deactivate' => sub ($self) {
        $self->active(0);
    };
};

# Use roles where needed
package MyApp::User;
use Moo;
with 'HasPermissions', 'Activatable';

has 'name' => (is => 'rw', required => 1);
```

## Module Testing Convention

Use standard Makefile.PL for distribution:

```perl
# Makefile.PL
use ExtUtils::MakeMaker;

WriteMakefile(
    NAME         => 'MyApp',
    VERSION_FROM => 'lib/MyApp.pm',
    ABSTRACT_FROM => 'lib/MyApp.pm',
    AUTHOR       => 'Your Name <you@example.com>',
    LICENSE      => 'perl',
    PREREQ_PM    => {
        'strict'     => 0,
        'warnings'   => 0,
        'Moo'        => '2.0',
        'Try::Tiny'  => '0.20',
    },
    TEST_REQUIRES => {
        'Test::More' => '0.96',
    },
    EXE_FILES    => [],
);
```

### Standard Test Command

```bash
perl Makefile.PL
make
make test
make install
```

## Module Structure

Typical module layout:

```
MyApp/
├── lib/
│   ├── MyApp.pm                 # Main module
│   └── MyApp/
│       ├── Database.pm          # Submodules
│       ├── User.pm
│       ├── Utils.pm
│       └── Exception.pm
├── t/
│   ├── 00-load.t               # Load tests
│   ├── 10-database.t           # Feature tests
│   ├── 20-user.t
│   └── fixtures/               # Test data
│       └── sample.json
├── examples/
│   └── basic_usage.pl          # Usage examples
├── Makefile.PL                 # Installation config
├── README.md
├── LICENSE
└── MANIFEST                    # File list
```

## Private vs Public Methods

```perl
# ✅ GOOD - Clear public API
package MyApp::User;

# Public methods
sub new ($class, %args) { }
sub get_name ($self) { }
sub set_email ($self, $email) { }

# Private methods (prefix with underscore)
sub _validate_name ($name) { }
sub _normalize_email ($email) { }

# Indicate in documentation which are public
=head1 PUBLIC METHODS

=over 4

=item new(%args)

Constructor

=item get_name()

Get user's name

=item set_email($email)

Set user's email

=back

=head1 PRIVATE METHODS

The following methods are private and not part of the public API:

=over 4

=item _validate_name($name)

Internal validation

=item _normalize_email($email)

Internal normalization

=back

=cut
```

## Module Development Checklist

- [ ] Package declaration at top of file
- [ ] `use strict; use warnings;` present
- [ ] Version number declared with `our $VERSION`
- [ ] Module ends with `1;`
- [ ] Clear public vs private API (underscore prefix for private)
- [ ] Consistent accessor naming (get_*, set_*, or just property)
- [ ] Constructors use named parameters
- [ ] Constructors validate inputs
- [ ] POD documentation complete
- [ ] Exporter used for clean public API (if applicable)
- [ ] Object-oriented modules use Moo or similar
- [ ] No global state without documentation
- [ ] Module has comprehensive tests
- [ ] Dependencies listed in Makefile.PL
- [ ] README explains module purpose


---


[↑ Back to TOC](#table-of-contents)

---

## 11. Performance Considerations

## The Golden Rule: Profile Before Optimizing

> **"Premature optimization is the root of all evil."** — Donald Knuth

### Why Profile First?

- **Humans are bad at guessing** where bottlenecks are
- **90% of execution time** is typically spent in 10% of code
- Optimization efforts are expensive and can introduce bugs
- Clear, maintainable code is easier to optimize later

```perl
# ❌ BAD - Optimizing without evidence
sub process_items {
    my (@items) = @_;
    
    # "Optimizing" by caching length (unnecessary)
    my $len = scalar @items;
    for (my $i = 0; $i < $len; $i++) {
        process($items[$i]);
    }
}

# ✅ GOOD - Clear code first, optimize if profiling shows need
sub process_items {
    my (@items) = @_;
    
    for my $item (@items) {
        process($item);
    }
}
```

## Benchmarking with Benchmark Module

The `Benchmark` module helps compare different approaches to solving the same problem.

### Basic Benchmarking

```perl
use Benchmark qw(timethis timethese cmpthese);
use strict;
use warnings;

# Test a single approach
timethis(1000000, sub {
    my $sum = 0;
    $sum += $_ for (1..100);
});

# Output: timethis 1000000:  2 wallclock secs...
```

### Comparing Multiple Approaches

```perl
use Benchmark qw(cmpthese);

# Compare different ways to concatenate strings
my @strings = qw(foo bar baz qux quux);

cmpthese(100000, {
    'join' => sub {
        my $result = join('', @strings);
    },
    'concat' => sub {
        my $result = '';
        $result .= $_ for @strings;
    },
    'interpolate' => sub {
        my $result = "@strings";
        $result =~ s/\s+//g;
    },
});

# Output shows relative performance:
#              Rate interpolate    concat      join
# interpolate 5000/s          --       -50%      -75%
# concat     10000/s        100%         --      -50%
# join       20000/s        300%       100%        --
```

### Real-World Example: Hash vs Array Lookup

```perl
use Benchmark qw(cmpthese);

my @array = (1..1000);
my %hash = map { $_ => 1 } @array;
my $search = 500;

cmpthese(50000, {
    'array_grep' => sub {
        my $found = grep { $_ == $search } @array;
    },
    'hash_exists' => sub {
        my $found = exists $hash{$search};
    },
});

# Result: Hash lookup is O(1), array grep is O(n)
#              Rate array_grep hash_exists
# array_grep  100/s         --        -98%
# hash_exists 5000/s      4900%          --
```

### Benchmarking Best Practices

```perl
# ✅ GOOD - Isolate the code you're testing
use Benchmark qw(cmpthese);

# Setup once, outside benchmark
my $data = load_test_data();

cmpthese(10000, {
    'method_a' => sub {
        process_method_a($data);  # Only benchmark this
    },
    'method_b' => sub {
        process_method_b($data);  # Only benchmark this
    },
});

# ❌ BAD - Including setup in benchmark
cmpthese(10000, {
    'method_a' => sub {
        my $data = load_test_data();  # Measured incorrectly
        process_method_a($data);
    },
});
```

## Profiling with Devel::NYTProf

`Devel::NYTProf` is the gold standard for Perl profiling. It shows exactly where your program spends time.

### Installation

```bash
cpanm Devel::NYTProf
# or
cpan Devel::NYTProf
```

### Running the Profiler

```bash
# Profile your script
perl -d:NYTProf your_script.pl

# Profile with arguments
perl -d:NYTProf your_script.pl --input data.txt

# Profile tests
perl -d:NYTProf -Ilib t/your_test.t

# This creates: nytprof.out
```

### Generating HTML Report

```bash
nytprofhtml

# Creates nytprof/ directory with HTML files
# Open nytprof/index.html in browser
```

### Interpreting Results

The HTML report shows:

1. **Time spent per subroutine** (inclusive and exclusive)
2. **Number of calls** to each subroutine
3. **Time per line** of code
4. **Call graph** showing what calls what

```perl
# Example code with profiling annotations
sub process_users {
    my @users = fetch_users();        # Line took 2.5s, 45% of total
    
    for my $user (@users) {
        validate_user($user);         # Called 10000 times, 0.1s total
        transform_data($user);        # Called 10000 times, 3.2s total ← BOTTLENECK
        save_user($user);             # Called 10000 times, 0.8s total
    }
}

# Profiling reveals transform_data is the bottleneck (3.2s / 6.6s total)
```

### Command Line Report

```bash
# Quick text summary
nytprofcsv

# CSV output for analysis
nytprofcsv --file=nytprof.out

# Merge multiple profile runs
nytprofmerge --out=merged.out nytprof.out.1 nytprof.out.2
```

### Profiling Best Practices

```perl
# ✅ GOOD - Profile with realistic data
# Use production-sized datasets
# Test typical usage patterns
# Profile complete workflows, not isolated functions

# ❌ BAD - Profiling unrealistic scenarios
# Don't profile with tiny test data
# Don't profile only edge cases
# Don't profile code in isolation from its context
```

## Common Performance Optimizations

### 1. Pre-compile Regular Expressions with `qr//`

```perl
# ❌ BAD - Regex recompiled every iteration
for my $line (@lines) {
    if ($line =~ /^\d{4}-\d{2}-\d{2}/) {
        process_date($line);
    }
}

# ✅ GOOD - Compile regex once
my $date_pattern = qr/^\d{4}-\d{2}-\d{2}/;
for my $line (@lines) {
    if ($line =~ $date_pattern) {
        process_date($line);
    }
}

# ✅ BETTER - For really hot loops
my $date_pattern = qr/^\d{4}-\d{2}-\d{2}/o;  # 'o' flag for compile-once
```

### 2. Choose Appropriate Data Structures

```perl
# ❌ BAD - Using array for membership tests
my @valid_codes = qw(200 201 204 301 302 304 400 404 500);

sub is_valid_code {
    my ($code) = @_;
    return grep { $_ == $code } @valid_codes;  # O(n) lookup
}

# ✅ GOOD - Use hash for O(1) lookup
my %valid_codes = map { $_ => 1 } qw(200 201 204 301 302 304 400 404 500);

sub is_valid_code {
    my ($code) = @_;
    return exists $valid_codes{$code};  # O(1) lookup
}

# Real impact: 100x faster for large sets
```

### 3. Avoid Unnecessary Copying

```perl
# ❌ BAD - Copies entire array
sub process_items {
    my (@items) = @_;  # Copies all elements
    
    for my $item (@items) {
        print $item->{name}, "\n";
    }
}

# ✅ GOOD - Pass by reference
sub process_items {
    my ($items_ref) = @_;
    
    for my $item (@$items_ref) {
        print $item->{name}, "\n";
    }
}

# Call it
process_items(\@large_array);  # No copy, just reference
```

### 4. Cache Expensive Computations

```perl
# ❌ BAD - Recalculating every time
sub get_user_permissions {
    my ($user_id) = @_;
    
    # Expensive database queries every call
    my $user = fetch_user($user_id);
    my @roles = fetch_roles($user_id);
    my @permissions = calculate_permissions(@roles);
    
    return @permissions;
}

# ✅ GOOD - Cache with Memoize
use Memoize;
memoize('get_user_permissions');

sub get_user_permissions {
    my ($user_id) = @_;
    
    my $user = fetch_user($user_id);
    my @roles = fetch_roles($user_id);
    my @permissions = calculate_permissions(@roles);
    
    return @permissions;
}

# ✅ BETTER - Manual cache with expiration
my %permission_cache;
my $cache_ttl = 300;  # 5 minutes

sub get_user_permissions {
    my ($user_id) = @_;
    
    my $now = time();
    my $cached = $permission_cache{$user_id};
    
    if ($cached && ($now - $cached->{time}) < $cache_ttl) {
        return @{ $cached->{permissions} };
    }
    
    # Compute fresh
    my $user = fetch_user($user_id);
    my @roles = fetch_roles($user_id);
    my @permissions = calculate_permissions(@roles);
    
    $permission_cache{$user_id} = {
        time        => $now,
        permissions => \@permissions,
    };
    
    return @permissions;
}
```

### 5. Use Built-in Functions

```perl
# ❌ BAD - Manual implementation
sub my_sum {
    my $total = 0;
    $total += $_ for @_;
    return $total;
}

# ✅ GOOD - Use optimized built-in
use List::Util qw(sum);
my $total = sum(@numbers);

# Other useful List::Util functions:
use List::Util qw(min max first any all none);

my $min = min(@numbers);
my $max = max(@numbers);
my $found = first { $_ > 100 } @numbers;
my $has_even = any { $_ % 2 == 0 } @numbers;
```

### 6. Avoid String Concatenation in Loops

```perl
# ❌ BAD - String grows with each iteration
my $output = '';
for my $item (@items) {
    $output .= $item->{name} . "\n";  # Reallocates each time
}

# ✅ GOOD - Build array, join once
my @lines;
for my $item (@items) {
    push @lines, $item->{name};
}
my $output = join("\n", @lines) . "\n";

# ✅ BETTER - Use map
my $output = join("\n", map { $_->{name} } @items) . "\n";
```

### 7. Use Lexical File Handles and Three-Arg Open

```perl
# ❌ BAD - Old style, slower
open(FILE, "< $filename") or die $!;
my @lines = <FILE>;
close(FILE);

# ✅ GOOD - Modern style, faster
open my $fh, '<', $filename or die "Cannot open $filename: $!";
my @lines = <$fh>;
close $fh;

# ✅ BETTER - Let Perl handle closing
my @lines = do {
    open my $fh, '<', $filename or die "Cannot open $filename: $!";
    <$fh>;
};
```

### 8. Lazy Loading and On-Demand Compilation

```perl
# ❌ BAD - Loading all modules upfront
use JSON;
use YAML;
use XML::Simple;
use Data::Dumper;
# All loaded even if not used

sub parse_data {
    my ($data, $format) = @_;
    
    if ($format eq 'json') {
        return decode_json($data);
    }
    # etc.
}

# ✅ GOOD - Load on demand
sub parse_data {
    my ($data, $format) = @_;
    
    if ($format eq 'json') {
        require JSON;
        return JSON::decode_json($data);
    } elsif ($format eq 'yaml') {
        require YAML;
        return YAML::Load($data);
    }
}
```

## When to Optimize vs. Premature Optimization

### ✅ When to Optimize

1. **After profiling identifies a bottleneck**
   - Specific function taking >10% of runtime
   - Obvious performance problem in production

2. **Known algorithmic issues**
   - O(n²) algorithm when O(n) exists
   - Nested loops over large datasets

3. **Resource constraints**
   - Memory usage causing swapping
   - CPU usage affecting other services

4. **User-facing performance issues**
   - Page load times >3 seconds
   - API response times >500ms

```perl
# ✅ GOOD - Justified optimization
# Profiling showed this function takes 80% of runtime
sub find_duplicates {
    my ($items_ref) = @_;
    
    # Before: O(n²) nested loop
    # After: O(n) with hash
    my %seen;
    my @duplicates;
    
    for my $item (@$items_ref) {
        if ($seen{$item}++) {
            push @duplicates, $item;
        }
    }
    
    return @duplicates;
}
```

### ❌ Premature Optimization (Avoid)

1. **Optimizing without measuring**
   - "This might be slow"
   - "Let me make this clever"

2. **Micro-optimizations**
   - Shaving microseconds off non-critical code
   - Trading readability for tiny gains

3. **Optimizing rarely-called code**
   - Startup routines
   - Configuration parsing
   - One-time initialization

```perl
# ❌ BAD - Premature optimization
# This runs once at startup, not worth the complexity
sub load_config {
    my ($file) = @_;
    
    # Overly complex "optimization" for one-time operation
    my %config;
    open my $fh, '<', $file or die $!;
    local $/ = undef;
    my $content = <$fh>;
    close $fh;
    
    # Complex parsing for marginal gain
    %config = map { split /=/, $_, 2 } split /\n/, $content;
    
    return \%config;
}

# ✅ GOOD - Clear and simple
sub load_config {
    my ($file) = @_;
    
    my %config;
    open my $fh, '<', $file or die "Cannot open $file: $!";
    
    while (my $line = <$fh>) {
        chomp $line;
        next if $line =~ /^\s*#/ || $line =~ /^\s*$/;
        
        if ($line =~ /^(\w+)\s*=\s*(.+)$/) {
            $config{$1} = $2;
        }
    }
    
    close $fh;
    return \%config;
}
```

## Performance Optimization Workflow

### Step-by-Step Process

1. **Identify the problem**
   - User reports or monitoring alerts
   - Specific slow operation
   - Quantify: "Takes 30s, should be <1s"

2. **Profile with Devel::NYTProf**
   ```bash
   perl -d:NYTProf slow_script.pl
   nytprofhtml
   # Open nytprof/index.html
   ```

3. **Find the bottleneck**
   - Look for functions with high exclusive time
   - Check number of calls (maybe called too often?)
   - Examine line-by-line time

4. **Analyze the cause**
   - Inefficient algorithm?
   - Unnecessary work?
   - Poor data structure choice?
   - Repeated expensive operations?

5. **Plan optimization**
   - Research better algorithms
   - Consider caching
   - Look for built-in solutions

6. **Implement change**
   - Keep original code commented for comparison
   - Add comment explaining optimization

7. **Benchmark improvement**
   ```perl
   use Benchmark qw(cmpthese);
   
   cmpthese(1000, {
       'original' => sub { original_implementation() },
       'optimized' => sub { optimized_implementation() },
   });
   ```

8. **Profile again**
   - Ensure optimization worked
   - Check for new bottlenecks
   - Verify memory usage didn't increase

9. **Test thoroughly**
   - Run full test suite
   - Test edge cases
   - Verify correctness

10. **Document**
    - Explain what was optimized and why
    - Include benchmark results in comments
    - Note any trade-offs

### Example Workflow

```perl
# Step 1: Identified slow user lookup (30s for 10k users)
# Step 2-3: Profiling showed fetch_user_data called 10k times (28s)
# Step 4: Each call hits database, no caching
# Step 5: Plan to add caching layer

# BEFORE: 30 seconds (from profiling)
sub process_users {
    my ($user_ids_ref) = @_;
    
    for my $id (@$user_ids_ref) {
        my $user = fetch_user_data($id);  # DB hit each time
        process_user($user);
    }
}

# AFTER: 2 seconds (benchmark result)
# Optimization: Batch fetch + cache
sub process_users {
    my ($user_ids_ref) = @_;
    
    # Batch fetch all users at once
    my $users = fetch_users_batch($user_ids_ref);
    
    for my $id (@$user_ids_ref) {
        process_user($users->{$id});
    }
}

sub fetch_users_batch {
    my ($ids_ref) = @_;
    
    my $placeholders = join(',', ('?') x @$ids_ref);
    my $sql = "SELECT * FROM users WHERE id IN ($placeholders)";
    
    my $sth = $dbh->prepare($sql);
    $sth->execute(@$ids_ref);
    
    my %users;
    while (my $row = $sth->fetchrow_hashref) {
        $users{$row->{id}} = $row;
    }
    
    return \%users;
}

# Result: 15x speedup (30s → 2s)
# Trade-off: Slightly higher memory usage (acceptable)
```

## Performance Checklist

Before claiming code is "optimized":

- [ ] **Profiled** the code to identify actual bottlenecks
- [ ] **Benchmarked** before and after changes
- [ ] **Documented** what was optimized and the performance gain
- [ ] **Tested** that optimized code produces correct results
- [ ] **Verified** memory usage hasn't significantly increased
- [ ] **Checked** that optimization doesn't hurt readability excessively
- [ ] **Considered** maintainability impact
- [ ] **Reviewed** algorithm complexity (O(n) vs O(n²), etc.)

### Quick Performance Wins

- [ ] Use hash lookups instead of array searches
- [ ] Pre-compile regexes outside loops with `qr//`
- [ ] Cache expensive function results with `Memoize`
- [ ] Pass large data structures by reference
- [ ] Use `join()` instead of concatenation in loops
- [ ] Leverage List::Util for built-in optimized functions
- [ ] Avoid unnecessary copying of arrays/hashes
- [ ] Use lexical filehandles and three-arg open
- [ ] Batch database queries instead of N+1 queries
- [ ] Load modules on-demand with `require` when appropriate

### Performance Anti-Patterns to Avoid

- [ ] Optimizing without profiling first
- [ ] Sacrificing readability for micro-optimizations
- [ ] Optimizing cold code (rarely executed)
- [ ] Using obscure tricks that confuse maintainers
- [ ] Ignoring algorithmic complexity (optimizing O(n²) instead of fixing)
- [ ] Premature optimization during initial development
- [ ] Not documenting performance-critical code


---


[↑ Back to TOC](#table-of-contents)

---

## 12. Code Quality Tools

## Why Quality Tools Matter

Quality tools automate the enforcement of best practices, catching issues before code review:

- **Consistency**: Automated formatting prevents style debates
- **Bug prevention**: Static analysis catches common mistakes
- **Test coverage**: Identifies untested code paths
- **Time savings**: Catches issues instantly, not in review
- **Learning**: Tools educate developers on best practices

> **"Let tools do the grunt work, humans do the thinking."**

## Perl::Critic - Static Code Analysis

`Perl::Critic` is the standard linter for Perl, enforcing community best practices.

### Installation

```bash
cpanm Perl::Critic
# or
cpan Perl::Critic
```

### Basic Usage

```bash
# Analyze a single file
perlcritic script.pl

# Analyze directory recursively
perlcritic lib/

# Show only severe violations
perlcritic --severity 5 lib/

# Verbose output with explanations
perlcritic --verbose 8 script.pl

# Count violations by severity
perlcritic --count lib/
```

### Understanding Severity Levels

Perl::Critic uses 5 severity levels (from least to most severe):

**Level 1 (Gentle)**: Cosmetic issues, style preferences
- Example: `grep in void context`
- Fix when time permits

**Level 2 (Stern)**: Stylistic but important
- Example: `postfix unless/if on multi-line statement`
- Should fix in new code

**Level 3 (Harsh)**: Potential bugs or maintainability issues
- Example: `bareword filehandle`
- Fix before committing

**Level 4 (Cruel)**: Likely bugs or serious maintainability problems
- Example: `two-argument open()`
- Fix immediately

**Level 5 (Brutal)**: Critical bugs or security issues
- Example: `using $a/$b outside sort`
- Must fix before deployment

```bash
# Start strict: Only show level 5 (critical)
perlcritic --severity 5 lib/

# Progress to level 3 (recommended minimum for new code)
perlcritic --severity 3 lib/

# Aim for level 1 (all violations) in mature projects
perlcritic --severity 1 lib/
```

### Common Policies

#### RequireUseStrict and RequireUseWarnings

```perl
# ❌ BAD - Violates policy
sub calculate_total {
    my $sum = 0;
    return $sum;
}

# ✅ GOOD - Satisfies policy
use strict;
use warnings;

sub calculate_total {
    my $sum = 0;
    return $sum;
}
```

#### ProhibitBarewordFilehandles

```perl
# ❌ BAD - Bareword filehandle (severity 5)
open(FILE, '<', $filename) or die $!;
my @lines = <FILE>;
close(FILE);

# ✅ GOOD - Lexical filehandle
open my $fh, '<', $filename or die "Cannot open $filename: $!";
my @lines = <$fh>;
close $fh;
```

#### RequireCheckedSyscalls

```perl
# ❌ BAD - Not checking return value (severity 4)
open my $fh, '<', $filename;
my $content = <$fh>;

# ✅ GOOD - Checking return value
open my $fh, '<', $filename or die "Cannot open $filename: $!";
my $content = <$fh>;
```

#### ProhibitStringyEval

```perl
# ❌ BAD - String eval is dangerous (severity 5)
my $code = "print 'Hello'";
eval $code;

# ✅ GOOD - Block eval for exception handling
eval {
    risky_operation();
};
if ($@) {
    handle_error($@);
}
```

#### RequireCarping

```perl
# ❌ BAD - die in module (shows module location, not caller)
package MyModule;

sub process {
    my ($data) = @_;
    die "Invalid data" unless defined $data;
}

# ✅ GOOD - croak shows caller's location
package MyModule;
use Carp qw(croak);

sub process {
    my ($data) = @_;
    croak "Invalid data" unless defined $data;
}
```

### Configuration File: `.perlcriticrc`

Create a `.perlcriticrc` file in your project root:

```ini
# Set default severity (1-5, lower = more strict)
severity = 3

# Only report violations of level 3 and above
# 1 = most strict, 5 = only critical

# Exclude specific policies
[-Subroutines::ProhibitExplicitReturnUndef]
# Reason: We prefer explicit "return undef" for clarity

[-ValuesAndExpressions::ProhibitMagicNumbers]
# Reason: Some magic numbers are self-documenting (0, 1, 100)

# Configure specific policies
[ValuesAndExpressions::ProhibitMagicNumbers]
allowed_values = 0 1 2 -1 100

[Subroutines::ProhibitManyArgs]
max_arguments = 5

[InputOutput::RequireCheckedSyscalls]
functions = open close print say

# Verbosity settings
verbose = %f:%l:%c: [%p] %m (Severity: %s)\n

# Theme: focus on specific areas
# theme = bugs || security || complexity
```

### Progressive Adoption Strategy

**Phase 1: Critical Issues (Week 1)**
```ini
# .perlcriticrc
severity = 5
```
Fix all severity 5 violations first.

**Phase 2: Likely Bugs (Week 2-3)**
```ini
severity = 4
```
Gradually reduce severity.

**Phase 3: Best Practices (Month 2)**
```ini
severity = 3
```
Level 3 is recommended for all production code.

**Phase 4: Style Polish (Ongoing)**
```ini
severity = 1
```
Aim for clean slate on new code.

### Disabling Specific Violations

```perl
# Disable for one line
my $dynamic_var = eval $code;  ## no critic (ProhibitStringyEval)

# Disable for block
## no critic (RequireCheckedSyscalls)
print "Quick debug output\n";
print "No need to check these\n";
## use critic

# Disable multiple policies
sub legacy_function {  ## no critic (ProhibitManyArgs, ProhibitExcessComplexity)
    my ($arg1, $arg2, $arg3, $arg4, $arg5, $arg6) = @_;
    # Complex legacy code here
}
```

**⚠ Warning**: Use `## no critic` sparingly and with justification. Over-use defeats the purpose.

## Perl::Tidy - Code Formatting

`Perl::Tidy` automatically formats code to consistent style standards.

### Installation

```bash
cpanm Perl::Tidy
# or
cpan Perl::Tidy
```

### Basic Usage

```bash
# Format file in-place with backup
perltidy -b script.pl
# Creates script.pl.bak (original) and formats script.pl

# Output to stdout (preview)
perltidy script.pl

# Format to specific output file
perltidy -o formatted.pl script.pl

# Format all Perl files in directory
find lib -name '*.pm' -exec perltidy -b {} \;
```

### Configuration File: `.perltidyrc`

Create `.perltidyrc` in project root:

```ini
# Basic settings
--indent-columns=4          # 4 space indentation
--maximum-line-length=100   # 100 char line limit
--warning-output            # Show warnings
--standard-error-output     # Errors to stderr

# Brace style
--opening-brace-on-new-line  # Opening brace placement
--brace-tightness=1          # Space before opening brace
--block-brace-tightness=0    # Block braces on separate lines

# Whitespace
--paren-tightness=1          # my_func( $arg )
--square-bracket-tightness=1 # $array[ $i ]
--brace-tightness=1          # $hash{ $key }

# Vertical spacing
--want-blank-lines-before-subs=1  # Blank line before sub
--blank-lines-before-subs=1

# Comments
--indent-block-comments      # Indent block comments with code
--static-block-comments      # Keep block comments in place

# Line breaks
--cuddled-else               # } else {
--opening-brace-always-on-right  # if (...) {

# Continuation indentation
--continuation-indentation=2 # Indent continued lines by 2
```

### Recommended `.perltidyrc` for This Guide

```ini
# Matches our style guide (Section 2)
--indent-columns=4
--maximum-line-length=100
--continuation-indentation=2
--cuddled-else
--opening-brace-always-on-right
--paren-tightness=1
--square-bracket-tightness=1
--brace-tightness=1
--want-blank-lines-before-subs=1
--blank-lines-before-packages=1
```

### Before and After Example

```perl
# BEFORE: Inconsistent formatting
sub calculate_total{
my($items)=@_;
my $total=0;
for my $item(@$items){
$total+=$item->{price}*$item->{quantity};
if($item->{discount}){
$total-=$item->{discount};}}
return $total;}

# AFTER: perltidy formatting
sub calculate_total {
    my ($items) = @_;
    my $total = 0;
    
    for my $item (@$items) {
        $total += $item->{price} * $item->{quantity};
        
        if ($item->{discount}) {
            $total -= $item->{discount};
        }
    }
    
    return $total;
}
```

### Integrating with Git Pre-commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit

# Format all staged Perl files
for file in $(git diff --cached --name-only --diff-filter=ACM | grep '\.p[lm]$')
do
    perltidy -b "$file"
    git add "$file"
    rm "${file}.bak"  # Remove backup
done
```

### Editor Integration

**VSCode**: Install Perl extension and configure:
```json
{
    "perl.perltidyrc": ".perltidyrc",
    "perl.enableFormat": true,
    "[perl]": {
        "editor.formatOnSave": true
    }
}
```

**Vim**: Add to `.vimrc`:
```vim
autocmd FileType perl setlocal equalprg=perltidy
```

**Emacs**: Use `cperl-mode` with perltidy integration.

## Devel::Cover - Test Coverage Analysis

`Devel::Cover` measures which parts of your code are tested.

### Installation

```bash
cpanm Devel::Cover
# or
cpan Devel::Cover
```

### Basic Usage

```bash
# Run tests with coverage
cover -test

# This does:
# 1. Deletes old coverage data (cover_db/)
# 2. Runs tests: HARNESS_PERL_SWITCHES=-MDevel::Cover make test
# 3. Generates HTML report in cover_db/coverage.html

# Open report
open cover_db/coverage.html  # macOS
xdg-open cover_db/coverage.html  # Linux
start cover_db/coverage.html  # Windows
```

### Manual Process

```bash
# 1. Delete old coverage data
rm -rf cover_db

# 2. Run tests with coverage
HARNESS_PERL_SWITCHES=-MDevel::Cover prove -l t/

# 3. Generate report
cover

# 4. Generate specific format
cover -report html        # HTML report (default)
cover -report text        # Text summary
cover -report json        # JSON output
cover -report coveralls   # Coveralls.io format
```

### Understanding Coverage Metrics

Coverage report shows:

**Statement Coverage**: % of statements executed
```perl
sub calculate {
    my ($x) = @_;
    return $x * 2;  # Statement covered if test calls calculate()
}
```

**Branch Coverage**: % of decision branches taken
```perl
sub process {
    my ($value) = @_;
    
    if ($value > 0) {      # Branch 1: true
        return 'positive';
    } else {               # Branch 2: false
        return 'non-positive';
    }
}
# 100% branch coverage requires tests for both $value > 0 and $value <= 0
```

**Condition Coverage**: % of boolean sub-expressions evaluated
```perl
if ($x > 0 && $y > 0) {  # Need tests for: TT, TF, FT, FF
    # code
}
```

**Subroutine Coverage**: % of subroutines called
```perl
sub helper_function {  # Covered if called by any test
    # ...
}
```

**POD Coverage**: % of public functions documented
```perl
=head2 public_function

This function does X.

=cut

sub public_function {  # Marked as documented
    # ...
}
```

### Interpreting Results

```bash
---------------------------- ------ ------ ------ ------ ------ ------
File                           stmt   bran   cond    sub    pod   time
---------------------------- ------ ------ ------ ------ ------ ------
lib/MyApp/User.pm             95.0   87.5   66.7  100.0    n/a   45.2
lib/MyApp/Database.pm        100.0  100.0  100.0  100.0    n/a   32.1
lib/MyApp/Utils.pm            78.3   50.0   33.3   80.0    n/a   22.7
---------------------------- ------ ------ ------ ------ ------ ------
Total                         91.1   79.2   66.7   93.3    n/a  100.0
---------------------------- ------ ------ ------ ------ ------ ------
```

**Good coverage targets:**
- **Statement**: 80%+ (aim for 90%+)
- **Branch**: 70%+ (aim for 80%+)
- **Subroutine**: 90%+ (most subs should be tested)

**Low coverage indicates:**
- Untested error handling paths
- Dead code that can be removed
- Complex functions needing more test cases

### Excluding Code from Coverage

```perl
# Exclude debugging code
sub debug_dump {
    # uncoverable subroutine
    use Data::Dumper;
    print Dumper(@_);
}

# Exclude unreachable code
if ($condition) {
    return 'ok';
} else {
    # uncoverable statement
    die "This should never happen";
}

# Exclude entire file (at top)
# uncoverable file
```

### Coverage in CI/CD

```bash
# .github/workflows/test.yml
- name: Run tests with coverage
  run: cover -test

- name: Check coverage threshold
  run: |
    coverage=$(cover -report text | grep Total | awk '{print $10}' | sed 's/%//')
    if (( $(echo "$coverage < 80" | bc -l) )); then
      echo "Coverage $coverage% is below 80%"
      exit 1
    fi

- name: Upload to Coveralls
  run: cover -report coveralls
  env:
    COVERALLS_REPO_TOKEN: ${{ secrets.COVERALLS_TOKEN }}
```

### Coverage Best Practices

```perl
# ✅ GOOD - Test both success and failure paths
sub divide {
    my ($a, $b) = @_;
    die "Division by zero" if $b == 0;
    return $a / $b;
}

# Tests needed for 100% coverage:
# - divide(10, 2) -> success path, statement coverage
# - divide(10, 0) -> error path, branch coverage

# ❌ BAD - Only testing happy path
# Test file with only: divide(10, 2)
# Leaves error handling untested
```

## Integrating Tools into Development Workflow

### Pre-commit Workflow

```bash
#!/bin/bash
# .git/hooks/pre-commit

echo "Running pre-commit checks..."

# 1. Format code
echo "Formatting code with perltidy..."
for file in $(git diff --cached --name-only --diff-filter=ACM | grep '\.p[lm]$'); do
    perltidy -b "$file"
    git add "$file"
    rm "${file}.bak"
done

# 2. Run Perl::Critic (severity 3+)
echo "Running Perl::Critic..."
if ! perlcritic --severity 3 --quiet $(git diff --cached --name-only --diff-filter=ACM | grep '\.p[lm]$'); then
    echo "Perl::Critic found violations. Fix before committing."
    exit 1
fi

# 3. Run tests
echo "Running tests..."
if ! prove -l t/; then
    echo "Tests failed. Fix before committing."
    exit 1
fi

echo "All checks passed!"
```

### Makefile Integration

```makefile
# Makefile

.PHONY: test critic tidy coverage quality

# Run all quality checks
quality: tidy critic test coverage

# Format code
tidy:
	find lib -name '*.pm' -exec perltidy -b {} \;
	find t -name '*.t' -exec perltidy -b {} \;
	find bin -type f -exec perltidy -b {} \;
	find . -name '*.bak' -delete

# Static analysis
critic:
	perlcritic --severity 3 lib/ t/ bin/

# Run tests
test:
	prove -l t/

# Coverage analysis
coverage:
	cover -test
	@echo "Coverage report: cover_db/coverage.html"
```

### CI/CD Pipeline

```yaml
# .github/workflows/perl.yml
name: Perl CI

on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Perl
      uses: shogo82148/actions-setup-perl@v1
      with:
        perl-version: '5.36'
    
    - name: Install dependencies
      run: |
        cpanm --installdeps .
        cpanm Perl::Critic Perl::Tidy Devel::Cover
    
    - name: Run Perl::Critic
      run: perlcritic --severity 3 lib/
    
    - name: Run tests
      run: prove -l t/
    
    - name: Coverage
      run: cover -test
    
    - name: Check coverage threshold
      run: |
        coverage=$(cover -report text | grep Total | awk '{print $10}' | sed 's/%//')
        echo "Coverage: $coverage%"
        if (( $(echo "$coverage < 80" | bc -l) )); then
          echo "Coverage below 80%"
          exit 1
        fi
```

### Editor Configuration

**VSCode** (`.vscode/settings.json`):
```json
{
    "perl.critic.severity": 3,
    "perl.critic.enabled": true,
    "perl.enableFormat": true,
    "perl.perltidyrc": ".perltidyrc",
    "[perl]": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "perl"
    }
}
```

## Quality Tools Checklist

### Project Setup

- [ ] `.perlcriticrc` configured with severity 3+ for new code
- [ ] `.perltidyrc` configured to match team style guide
- [ ] `cover -test` runs successfully with >80% coverage
- [ ] Pre-commit hooks installed for formatting and linting
- [ ] CI/CD pipeline runs all quality checks
- [ ] Make/build targets for quality checks (`make quality`)

### Daily Development

- [ ] Run `perltidy` before committing (or auto-format on save)
- [ ] Check `perlcritic` violations (aim for zero at severity 3+)
- [ ] Run tests with coverage for changed files
- [ ] Review coverage report for new code (aim for 90%+)
- [ ] Fix all severity 5 and 4 violations immediately
- [ ] Document reasons for any `## no critic` exceptions

### Code Review

- [ ] Verify code passes `perlcritic --severity 3`
- [ ] Check formatting is consistent (perltidy was run)
- [ ] Review test coverage for changed files
- [ ] Question any `## no critic` pragmas
- [ ] Ensure new code has tests (statement + branch coverage)
- [ ] Validate error paths are tested

### Regular Maintenance

- [ ] Monthly: Review and tighten `.perlcriticrc` policies
- [ ] Quarterly: Audit `## no critic` usage (is it still needed?)
- [ ] Track coverage trends (should increase or stay stable)
- [ ] Update tooling versions (`cpanm --self-upgrade`)
- [ ] Review new Perl::Critic policies for adoption


---


[↑ Back to TOC](#table-of-contents)

---

## 13. Common Anti-Patterns

## What Are Anti-patterns?

Anti-patterns are common solutions that initially seem reasonable but lead to:

- **Bugs**: Subtle errors that appear under specific conditions
- **Maintenance nightmares**: Code that's hard to understand or modify
- **Performance issues**: Inefficient implementations
- **Security vulnerabilities**: Exploitable weaknesses

> **"The best way to learn is from other people's mistakes."**

This section catalogs common Perl anti-patterns and their better alternatives.

---

## Using $a and $b Outside of Sort

### The Problem

`$a` and `$b` are special package variables reserved for `sort`. Using them elsewhere causes:
- Name collisions with sort's internal variables
- Subtle bugs in nested sorts
- Confusion for maintainers
- No warnings from `use strict`

### ❌ BAD Example

```perl
use strict;
use warnings;

sub compare_values {
    my ($a, $b) = @_;  # Dangerous! Shadows sort's $a/$b
    return $a <=> $b;
}

my @numbers = (5, 2, 8, 1);
my @sorted = sort { compare_values($a, $b) } @numbers;  # Bug!
# $a and $b inside compare_values are NOT the same as sort's $a/$b
```

### ✅ GOOD Alternative

```perl
use strict;
use warnings;

sub compare_values {
    my ($x, $y) = @_;  # Use different names
    return $x <=> $y;
}

my @numbers = (5, 2, 8, 1);
my @sorted = sort { compare_values($a, $b) } @numbers;  # Works correctly
```

### Why This Happens

```perl
# $a and $b are package globals, not lexical
package main;
our ($a, $b);  # Implicitly declared by Perl for sort

# Using them as parameters doesn't create new lexicals
sub bad_function {
    my ($a, $b) = @_;  # Assigns to package $main::a, $main::b
    # Not new lexical variables!
}
```

### ✅ Better: Always Use Descriptive Names

```perl
# Clear and safe
sub compare_users_by_age {
    my ($user1, $user2) = @_;
    return $user1->{age} <=> $user2->{age};
}

my @sorted_users = sort { compare_users_by_age($a, $b) } @users;
```

---

## Bareword Filehandles

### The Problem

Bareword filehandles (e.g., `FILE`, `FH`) are:
- Global to the package
- Not automatically closed when they go out of scope
- Prone to name collisions
- Deprecated in modern Perl

### ❌ BAD Example

```perl
sub read_config {
    my ($filename) = @_;
    
    open(FILE, '<', $filename) or die $!;  # Bareword filehandle
    my @lines = <FILE>;
    close(FILE);
    
    return @lines;
}

# Problem: FILE is global, can conflict with other uses
```

### ✅ GOOD Alternative

```perl
sub read_config {
    my ($filename) = @_;
    
    open my $fh, '<', $filename or die "Cannot open $filename: $!";
    my @lines = <$fh>;
    close $fh;  # Optional: auto-closes when $fh goes out of scope
    
    return @lines;
}
```

### Why Lexical Filehandles Are Better

```perl
# Lexical filehandles auto-close
sub process_file {
    my ($filename) = @_;
    
    {
        open my $fh, '<', $filename or die $!;
        my $content = <$fh>;
        # $fh automatically closed when leaving this block
    }
    
    # File is already closed here
}

# Can't accidentally reuse handle
sub safe_nested_reads {
    open my $fh1, '<', 'file1.txt' or die $!;
    {
        open my $fh2, '<', 'file2.txt' or die $!;
        # Both handles coexist safely
    }
}
```

---

## Not Checking open() Return Values

### The Problem

Failing to check `open()` return values causes:
- Silent failures (file doesn't exist, permissions denied)
- Operations on undefined filehandles
- Confusing error messages later in execution
- Data loss or corruption

### ❌ BAD Example

```perl
sub save_data {
    my ($filename, $data) = @_;
    
    open my $fh, '>', $filename;  # No error checking!
    print $fh $data;
    close $fh;
    
    # If open failed, $fh is undefined, print fails silently
}
```

### ✅ GOOD Alternative

```perl
sub save_data {
    my ($filename, $data) = @_;
    
    open my $fh, '>', $filename 
        or die "Cannot open $filename for writing: $!";
    
    print $fh $data 
        or die "Cannot write to $filename: $!";
    
    close $fh 
        or die "Cannot close $filename: $!";
}
```

### ✅ BETTER: Use autodie

```perl
use autodie;  # Automatically die on failed system calls

sub save_data {
    my ($filename, $data) = @_;
    
    open my $fh, '>', $filename;  # autodie handles error
    print $fh $data;
    close $fh;
}
```

### Real-World Impact

```perl
# ❌ BAD - Silent failure
sub log_message {
    my ($msg) = @_;
    open my $fh, '>>', '/var/log/app.log';  # Might fail (permissions)
    print $fh "$msg\n";
    close $fh;
    # User thinks message was logged, but it wasn't!
}

# ✅ GOOD - Fail fast with clear error
sub log_message {
    my ($msg) = @_;
    
    open my $fh, '>>', '/var/log/app.log'
        or die "Cannot append to log: $!";
    
    print $fh "$msg\n"
        or die "Cannot write to log: $!";
    
    close $fh
        or warn "Error closing log: $!";  # Warn instead of die for close
}
```

---

## Using eval for Flow Control

### The Problem

String `eval` is dangerous and slow:
- Injects code at runtime (security risk)
- Bypasses syntax checking
- Difficult to debug
- Performance overhead

Block `eval` is fine for exception handling, but not for flow control.

### ❌ BAD Example

```perl
# Using string eval for dynamic code
sub calculate {
    my ($operation, $x, $y) = @_;
    
    # Dangerous! User input could be malicious
    my $result = eval "$x $operation $y";
    return $result;
}

# calculate('*', 2, 3) -> 6
# calculate('; system("rm -rf /"); #', 2, 3) -> DISASTER!
```

### ✅ GOOD Alternative

```perl
# Use dispatch table
my %operations = (
    '+' => sub { $_[0] + $_[1] },
    '-' => sub { $_[0] - $_[1] },
    '*' => sub { $_[0] * $_[1] },
    '/' => sub { $_[0] / $_[1] },
);

sub calculate {
    my ($operation, $x, $y) = @_;
    
    my $op = $operations{$operation}
        or die "Unknown operation: $operation";
    
    return $op->($x, $y);
}
```

### ❌ BAD: eval for Flow Control

```perl
# Using eval to suppress errors (wrong!)
sub get_value {
    my ($key) = @_;
    
    my $value = eval { $config->{$key}->{nested}->{deep} };
    return $value // 'default';
}
```

### ✅ GOOD: Proper Checking

```perl
sub get_value {
    my ($key) = @_;
    
    return 'default' unless exists $config->{$key};
    return 'default' unless ref $config->{$key} eq 'HASH';
    return 'default' unless exists $config->{$key}->{nested};
    # ... proper validation
    
    return $config->{$key}->{nested}->{deep};
}

# ✅ BETTER: Use Try::Tiny for actual exception handling
use Try::Tiny;

try {
    risky_operation();
} catch {
    handle_error($_);
};
```

---

## Deep Nesting (Arrow Anti-pattern)

### The Problem

Excessive nesting (>3 levels) creates:
- Hard-to-read code
- Arrow shape (`>>>>>>>`)
- Difficult testing and debugging
- High cyclomatic complexity

### ❌ BAD Example

```perl
sub process_order {
    my ($order_id) = @_;
    
    if (defined $order_id) {
        my $order = get_order($order_id);
        if (defined $order) {
            if ($order->{status} eq 'pending') {
                my $customer = get_customer($order->{customer_id});
                if (defined $customer) {
                    if ($customer->{credit} > $order->{total}) {
                        if (charge_customer($customer, $order->{total})) {
                            update_order_status($order_id, 'paid');
                            return 1;
                        }
                    }
                }
            }
        }
    }
    
    return 0;  # What went wrong? Hard to tell!
}
```

### ✅ GOOD: Guard Clauses (Early Returns)

```perl
sub process_order {
    my ($order_id) = @_;
    
    # Validate inputs first
    return 0 unless defined $order_id;
    
    my $order = get_order($order_id);
    return 0 unless defined $order;
    return 0 unless $order->{status} eq 'pending';
    
    # Process customer
    my $customer = get_customer($order->{customer_id});
    return 0 unless defined $customer;
    return 0 unless $customer->{credit} > $order->{total};
    
    # Execute transaction
    return 0 unless charge_customer($customer, $order->{total});
    
    update_order_status($order_id, 'paid');
    return 1;
}
```

### ✅ BETTER: Extract Helper Functions

```perl
sub process_order {
    my ($order_id) = @_;
    
    my $order = validate_order($order_id) or return 0;
    my $customer = validate_customer($order) or return 0;
    
    return execute_payment($order, $customer);
}

sub validate_order {
    my ($order_id) = @_;
    
    return unless defined $order_id;
    
    my $order = get_order($order_id);
    return unless defined $order;
    return unless $order->{status} eq 'pending';
    
    return $order;
}

sub validate_customer {
    my ($order) = @_;
    
    my $customer = get_customer($order->{customer_id});
    return unless defined $customer;
    return unless $customer->{credit} > $order->{total};
    
    return $customer;
}

sub execute_payment {
    my ($order, $customer) = @_;
    
    return 0 unless charge_customer($customer, $order->{total});
    
    update_order_status($order->{id}, 'paid');
    return 1;
}
```

---

## Global Variables Without Documentation

### The Problem

Global variables are:
- Hard to track and debug
- Source of action-at-a-distance bugs
- Difficult to test
- Often misused across modules

### ❌ BAD Example

```perl
package MyApp;
use strict;
use warnings;

our $DEBUG;        # What does this control?
our %CONFIG;       # Where is this set?
our $LAST_ERROR;   # Who sets this, when, why?

sub process_data {
    my ($data) = @_;
    
    print "Processing...\n" if $DEBUG;  # Global state
    
    if (error_occurred()) {
        $LAST_ERROR = "Processing failed";  # Side effect!
        return;
    }
}
```

### ✅ GOOD: Document Globals

```perl
package MyApp;
use strict;
use warnings;

=head1 PACKAGE GLOBALS

=head2 $DEBUG

Set to true to enable debug output. Default: 0

=cut

our $DEBUG = 0;

=head2 %CONFIG

Application configuration, set by load_config().
Do not modify directly after initialization.

=cut

our %CONFIG;

=head2 $LAST_ERROR

Most recent error message. Set by public methods on failure.
Check this after any method returns undef.

=cut

our $LAST_ERROR;
```

### ✅ BETTER: Avoid Globals

```perl
# Use object-oriented approach
package MyApp;

sub new {
    my ($class) = @_;
    
    return bless {
        debug      => 0,
        config     => {},
        last_error => undef,
    }, $class;
}

sub process_data {
    my ($self, $data) = @_;
    
    print "Processing...\n" if $self->{debug};
    
    if ($self->error_occurred()) {
        $self->{last_error} = "Processing failed";
        return;
    }
}

sub get_error {
    my ($self) = @_;
    return $self->{last_error};
}
```

---

## Prototypes Misuse

### The Problem

Perl prototypes are **not** like function signatures in other languages:
- Only affect bareword parsing
- Don't validate argument types or counts at runtime
- Confusing syntax
- Limited usefulness

### ❌ BAD Example

```perl
# Misunderstanding prototypes
sub add($$) {  # This does NOT validate two arguments!
    my ($a, $b) = @_;
    return $a + $b;
}

add(1);        # No error! $b is undef
add(1, 2, 3);  # No error! Third arg ignored
add('a', 'b'); # No error! No type checking
```

### What Prototypes Actually Do

```perl
# Prototype affects parsing of barewords
sub my_push (\@@) {  # First arg is array ref, rest are scalars
    my $array_ref = shift;
    push @$array_ref, @_;
}

my @arr = (1, 2, 3);
my_push(@arr, 4, 5);  # Works: @arr passed as reference automatically
# Equivalent to: my_push(\@arr, 4, 5)
```

### ✅ GOOD: Skip Prototypes, Validate Manually

```perl
use Carp qw(croak);

sub add {
    my ($a, $b) = @_;
    
    croak "add requires two arguments" unless @_ == 2;
    croak "First argument must be numeric" unless looks_like_number($a);
    croak "Second argument must be numeric" unless looks_like_number($b);
    
    return $a + $b;
}
```

### ✅ BETTER: Use Modern Signatures (Perl 5.36+)

```perl
use v5.36;  # Enables signatures

sub add($a, $b) {
    return $a + $b;
}

add(1);     # ERROR: Not enough arguments
add(1, 2, 3);  # ERROR: Too many arguments
```

---

## String Concatenation in Loops

### The Problem

Repeated string concatenation in loops:
- Reallocates memory each iteration
- O(n²) time complexity for n concatenations
- Slow for large strings
- High memory usage

### ❌ BAD Example

```perl
sub generate_html {
    my (@items) = @_;
    
    my $html = '';
    $html .= "<ul>\n";
    
    for my $item (@items) {
        $html .= "  <li>$item->{name}</li>\n";  # Realloc each time!
    }
    
    $html .= "</ul>\n";
    return $html;
}

# For 10,000 items, this is extremely slow
```

### ✅ GOOD: Build Array, Join Once

```perl
sub generate_html {
    my (@items) = @_;
    
    my @lines;
    push @lines, "<ul>";
    
    for my $item (@items) {
        push @lines, "  <li>$item->{name}</li>";
    }
    
    push @lines, "</ul>";
    return join("\n", @lines) . "\n";
}
```

### ✅ BETTER: Use map

```perl
sub generate_html {
    my (@items) = @_;
    
    my $items_html = join("\n", 
        map { "  <li>$_->{name}</li>" } @items
    );
    
    return "<ul>\n$items_html\n</ul>\n";
}
```

### Performance Comparison

```perl
use Benchmark qw(cmpthese);

my @items = map { {name => "Item $_"} } (1..10000);

cmpthese(100, {
    'concatenation' => sub {
        my $html = '';
        $html .= "<li>$_->{name}</li>" for @items;
    },
    'join' => sub {
        my @parts = map { "<li>$_->{name}</li>" } @items;
        my $html = join('', @parts);
    },
});

# Result: join is 10-20x faster for large lists
```

---

## Context-Insensitive Functions

### The Problem

Functions that ignore calling context can surprise callers:
- List context expects list
- Scalar context expects single value
- Ignoring context leads to bugs

### ❌ BAD Example

```perl
sub get_users {
    my @users = fetch_all_users();  # Always returns list
    return @users;  # Same in scalar or list context
}

# In scalar context, returns COUNT of users, not expected single user
my $user = get_users();  # $user = 42 (count!) - Bug!

# In list context, works as expected
my @users = get_users();  # @users = (user1, user2, ...) - OK
```

### ✅ GOOD: Context-Aware Return

```perl
sub get_users {
    my @users = fetch_all_users();
    
    if (wantarray) {
        # List context: return all users
        return @users;
    } else {
        # Scalar context: return count or first user
        return scalar @users;  # or return $users[0];
    }
}

my $count = get_users();   # Scalar context: count
my @all = get_users();     # List context: list
```

### ✅ BETTER: Explicit Functions

```perl
# Clear intent, no ambiguity
sub get_all_users {
    return fetch_all_users();  # Always returns list
}

sub get_user_count {
    my @users = fetch_all_users();
    return scalar @users;
}

sub get_first_user {
    my @users = fetch_all_users();
    return $users[0];
}

# Usage is clear
my @users = get_all_users();      # Obvious: list
my $count = get_user_count();     # Obvious: number
my $user = get_first_user();      # Obvious: single user
```

---

## Additional Anti-patterns

### Magic Numbers Without Constants

```perl
# ❌ BAD - What does 86400 mean?
if ($elapsed > 86400) {
    expire_session();
}

# ✅ GOOD - Use named constant
use constant SECONDS_PER_DAY => 86400;

if ($elapsed > SECONDS_PER_DAY) {
    expire_session();
}
```

### Unnecessary regex modifiers

```perl
# ❌ BAD - Case-insensitive when not needed
if ($status =~ /active/i) {  # Why case-insensitive?
    # ...
}

# ✅ GOOD - Be explicit about case sensitivity
if ($status eq 'active') {  # Exact match, no regex overhead
    # ...
}

# Use regex only when actually needed
if ($email =~ /@example\.com$/i) {  # Case-insensitive for domain
    # ...
}
```

### Modifying $_ in Unexpected Places

```perl
# ❌ BAD - Modifying $_ has side effects
sub process_items {
    for (@items) {
        $_ = transform($_);  # Modifies original @items!
    }
}

# ✅ GOOD - Use named variable
sub process_items {
    for my $item (@items) {
        $item = transform($item);  # Intent is clear
    }
}

# ✅ BETTER - Don't modify, return new list
sub process_items {
    return map { transform($_) } @items;
}
```

---

## Anti-patterns Checklist

Use this checklist during code review to catch common anti-patterns:

### Variable Usage
- [ ] No use of `$a`/`$b` outside `sort` blocks
- [ ] No bareword filehandles (use lexical: `my $fh`)
- [ ] No undocumented package globals
- [ ] No modification of `$_` outside `map`/`grep`/`for` where expected

### Error Handling
- [ ] All `open()` calls check return value or use `autodie`
- [ ] No string `eval` for flow control (use block eval or dispatch)
- [ ] All system calls check for errors
- [ ] `die`/`croak` messages include context (`$!`, filename, etc.)

### Code Structure
- [ ] No nesting deeper than 3 levels (use guard clauses)
- [ ] No functions longer than 50 lines (extract helpers)
- [ ] No functions with >5 arguments (use objects/hashes)
- [ ] No copy-pasted code blocks (DRY principle)

### Performance
- [ ] No string concatenation in loops (use `join`)
- [ ] No repeated regex compilation in loops (use `qr//`)
- [ ] No array `grep` for membership tests (use hash)
- [ ] No unnecessary copying of large data structures

### Style
- [ ] No magic numbers (use constants)
- [ ] No misleading function names (`get_user` returns single user, not array)
- [ ] Context-aware returns when appropriate (`wantarray`)
- [ ] No prototypes unless absolutely necessary (prefer signatures)

### Security
- [ ] No user input in string `eval`
- [ ] No system commands built with unvalidated input
- [ ] No SQL built with string concatenation (use placeholders)
- [ ] Taint mode enabled for setuid scripts


---


[↑ Back to TOC](#table-of-contents)

---

## 14. Resources & References

## Official Perl Documentation

Perl's built-in documentation (`perldoc`) is comprehensive and authoritative. Always start here.

### Essential perldoc Pages

```bash
# Introduction for beginners
perldoc perlintro

# Language syntax overview
perldoc perlsyn

# Built-in functions reference
perldoc perlfunc

# Variables and special variables
perldoc perlvar

# Regular expressions guide
perldoc perlre
perldoc perlrequick   # Quick start
perldoc perlretut     # Detailed tutorial

# References and data structures
perldoc perlref
perldoc perldsc
perldoc perllol

# Object-oriented programming
perldoc perlobj
perldoc perlboot      # Tutorial
perldoc perltoot      # Tom's OO Tutorial

# Modules and packages
perldoc perlmod
perldoc perlmodlib    # Standard library
perldoc perlmodinstall

# Debugging
perldoc perldebug
perldoc perldiag      # Warning/error messages

# Best practices and style
perldoc perlstyle
perldoc perlsec       # Security

# FAQ
perldoc perlfaq
perldoc perlfaq1      # through perlfaq9
```

### Quick perldoc Tips

```bash
# Search for function
perldoc -f print
perldoc -f map
perldoc -f grep

# Search for variable
perldoc -v '$@'
perldoc -v '$!'
perldoc -v '@INC'

# Search module documentation
perldoc Module::Name
perldoc DBI
perldoc Mojolicious

# Get source code of installed module
perldoc -m Module::Name

# Search all documentation
perldoc -q "regex"
perldoc -q "hash"
```

### Understanding perldoc Structure

```bash
# Core documentation index
perldoc perl

# Main sections:
# - Tutorials: perlintro, perltoot, perlboot
# - Reference: perlfunc, perlvar, perlsyn
# - Internals: perlguts, perlapi
# - Platform: perlwin32, perlaix
# - History: perlhist, perldelta
```

---

## Essential Books

### 1. Perl Best Practices by Damian Conway

**The definitive style guide for professional Perl development.**

**Key topics:**
- Code layout and naming conventions
- Subroutine design patterns
- Reference and data structure best practices
- Error handling strategies
- Testing methodologies
- Documentation standards

**Why read it:**
- Industry-standard recommendations
- Comprehensive coverage of Perl idioms
- Practical examples and rationale
- Foundation for Perl::Critic policies

**Best for:** Intermediate to advanced developers establishing team standards.

### 2. Modern Perl (4th Edition)

**Free online and in print, covers modern Perl 5.36+ features.**

**Key topics:**
- Modern Perl syntax and features
- Idiomatic Perl programming
- CPAN ecosystem
- Testing and deployment
- Web development with Perl

**Why read it:**
- Always up-to-date with latest Perl
- Free PDF/HTML version available
- Practical, opinionated advice
- Focuses on maintainable code

**Best for:** Developers transitioning to modern Perl practices.

**Website:** http://modernperlbooks.com/books/modern_perl_2016/

### 3. Learning Perl (8th Edition) by Randal Schwartz et al.

**The classic introductory Perl book, "The Llama Book."**

**Key topics:**
- Perl fundamentals
- Scalars, arrays, hashes
- Regular expressions basics
- File handling
- Subroutines and references

**Why read it:**
- Perfect for absolute beginners
- Clear explanations and exercises
- Builds solid foundation
- Trusted by generations of Perl programmers

**Best for:** Complete beginners to Perl programming.

### 4. Programming Perl (4th Edition) by Tom Christiansen et al.

**The comprehensive Perl reference, "The Camel Book."**

**Key topics:**
- Complete language reference
- Advanced techniques
- Unicode and internationalization
- Performance optimization
- Security considerations

**Why read it:**
- Authoritative reference by Perl creators
- Deep technical details
- Covers edge cases
- Essential for advanced users

**Best for:** Comprehensive reference for all skill levels.

### 5. Intermediate Perl (2nd Edition)

**Bridges the gap between beginner and advanced topics.**

**Key topics:**
- References and complex data structures
- Object-oriented programming
- Modules and distributions
- Testing strategies
- Advanced regex

**Why read it:**
- Natural progression after Learning Perl
- Practical real-world examples
- Excellent OO introduction
- Module creation guidance

**Best for:** Developers ready to go beyond basics.

### 6. Higher-Order Perl by Mark Jason Dominus

**Advanced functional programming techniques in Perl.**

**Key topics:**
- Recursion and memoization
- Iterators and streams
- Callbacks and closures
- Parser construction
- Declarative programming

**Why read it:**
- Mind-expanding programming concepts
- Applicable to any language
- Deep computer science foundations
- Free PDF available online

**Best for:** Advanced developers interested in functional programming.

**Website:** https://hop.perl.plover.com/

---

## CPAN: The Comprehensive Perl Archive Network

### What is CPAN?

CPAN is Perl's massive ecosystem of reusable modules:
- **Over 200,000 modules** from 40,000+ authors
- **Battle-tested** solutions to common problems
- **Well-documented** with examples and tests
- **Free and open source**

### Searching CPAN

**MetaCPAN** (https://metacpan.org/)
- Modern web interface
- Excellent search and filtering
- Documentation rendering
- Module ratings and reviews

```bash
# Command-line search
cpan -l Module::Name
cpan -D Module::Name  # Detailed info
```

### Popular Modules by Category

#### Web Development

```perl
# Web frameworks
use Mojolicious;      # Full-featured modern framework
use Catalyst;         # MVC framework for large applications
use Dancer2;          # Lightweight, expressive framework
use Plack;            # PSGI server interface

# Web utilities
use LWP::UserAgent;   # HTTP client
use HTTP::Tiny;       # Minimal HTTP client
use JSON;             # JSON encoding/decoding
use XML::LibXML;      # XML parsing
```

#### Database

```perl
# Database abstraction
use DBI;              # Database interface (essential)
use DBIx::Class;      # ORM (Object-Relational Mapper)
use DBD::mysql;       # MySQL driver
use DBD::Pg;          # PostgreSQL driver
use DBD::SQLite;      # SQLite driver
```

#### Data Structures and Utilities

```perl
# Data manipulation
use List::Util qw(sum min max first any all);
use List::MoreUtils qw(uniq zip);
use Data::Dumper;     # Data structure printing
use Clone qw(clone);  # Deep copying

# Date and time
use DateTime;         # Comprehensive date/time
use Time::Piece;      # Core module, simple date handling
use Date::Parse;      # Parse date strings
```

#### File and Path Handling

```perl
use File::Spec;       # Portable path operations
use Path::Tiny;       # Modern path manipulation
use File::Slurp;      # Read/write entire files
use File::Find::Rule; # Advanced file finding
```

#### Testing

```perl
use Test::More;       # Standard testing framework
use Test::Exception;  # Test exception handling
use Test::MockModule; # Mock modules for testing
use Test::Deep;       # Deep data structure comparison
```

#### Text Processing

```perl
use Text::CSV;        # CSV parsing/generation
use Template;         # Template Toolkit
use String::CamelCase;# Case conversion
use Regexp::Common;   # Common regex patterns
```

#### Object-Oriented Programming

```perl
use Moose;            # Modern OO framework
use Moo;              # Minimalist Moose-like OO
use Class::Accessor;  # Automatic accessor generation
use namespace::autoclean; # Clean up imports
```

#### Async and Parallel

```perl
use AnyEvent;         # Event-driven programming
use IO::Async;        # Asynchronous I/O
use Parallel::ForkManager; # Process management
use MCE;              # Many-Core Engine for parallelism
```

### Installing CPAN Modules

```bash
# Using cpanm (recommended)
cpanm Module::Name
cpanm --verbose Module::Name    # Show details
cpanm --force Module::Name      # Force install
cpanm --notest Module::Name     # Skip tests (faster)

# Using cpan
cpan Module::Name

# Using system package manager (safer for production)
# Ubuntu/Debian:
sudo apt-get install libmodule-name-perl

# RedHat/CentOS:
sudo yum install perl-Module-Name
```

### Managing Dependencies

**cpanfile** (like package.json or requirements.txt):

```perl
# cpanfile
requires 'Mojolicious', '>= 9.0';
requires 'DBI';
requires 'DBD::Pg', '>= 3.0';

on 'test' => sub {
    requires 'Test::More', '>= 1.0';
    requires 'Test::Exception';
};

on 'develop' => sub {
    requires 'Perl::Critic';
    requires 'Perl::Tidy';
};
```

```bash
# Install all dependencies
cpanm --installdeps .

# Install with specific phase
cpanm --installdeps --with-develop .
```

---

## Community Resources

### PerlMonks (https://perlmonks.org/)

**The premier Perl Q&A community.**

**Features:**
- Ask questions, get expert answers
- Code reviews and tutorials
- "Seekers of Perl Wisdom" section for questions
- "Cool Uses for Perl" for sharing projects
- Long history, deep knowledge base

**Best for:** Getting help with specific problems, learning from experts.

### Reddit r/perl (https://reddit.com/r/perl)

**Active community discussion.**

**Features:**
- News and announcements
- Show & Tell for projects
- Weekly challenges
- Career advice
- Modern, accessible platform

**Best for:** Staying current, sharing projects, casual discussion.

### blogs.perl.org

**Aggregated Perl blog posts.**

**Features:**
- Technical articles
- CPAN module announcements
- Community news
- Multiple authors and perspectives

**Best for:** In-depth articles, learning new techniques.

### Perl Weekly (https://perlweekly.com/)

**Email newsletter of Perl news and articles.**

**Features:**
- Curated weekly updates
- Module releases
- Blog posts and tutorials
- Jobs and events
- Free subscription

**Best for:** Staying informed without daily checking.

### IRC Channels

**Freenode/Libera Chat:**
- `#perl` - General discussion
- `#perl-help` - Beginner questions
- `#catalyst` - Catalyst framework
- `#mojolicious` - Mojolicious framework
- `#dbix-class` - DBIx::Class ORM

**Best for:** Real-time help and discussion.

### Perl Conferences

**The Perl and Raku Conference (formerly YAPC)**
- Annual conference in North America, Europe, Asia
- Talks, tutorials, networking
- Videos available online after conference

**Local Perl Mongers Groups**
- Find local user groups at https://www.pm.org/
- Regular meetups and technical talks

---

## Tools and Frameworks

### Web Frameworks

#### Mojolicious

**Modern, full-featured, real-time web framework.**

```perl
use Mojolicious::Lite;

get '/' => sub {
    my $c = shift;
    $c->render(text => 'Hello World!');
};

app->start;
```

**Features:**
- Non-blocking I/O
- WebSockets support
- Built-in testing
- No dependencies beyond core Perl
- Excellent documentation

**Best for:** Modern web applications, APIs, real-time apps.

**Website:** https://mojolicious.org/

#### Catalyst

**MVC framework for large applications.**

```perl
package MyApp::Controller::Root;
use Moose;
use namespace::autoclean;

BEGIN { extends 'Catalyst::Controller' }

sub index :Path :Args(0) {
    my ($self, $c) = @_;
    $c->response->body('Hello World');
}
```

**Features:**
- Full MVC architecture
- Powerful plugin system
- Mature and stable
- Excellent for large teams

**Best for:** Enterprise applications, large codebases.

**Website:** http://catalyst.perl.org/

#### Dancer2

**Lightweight, expressive web framework.**

```perl
use Dancer2;

get '/' => sub {
    return 'Hello World';
};

start;
```

**Features:**
- Sinatra-inspired syntax
- Easy to learn
- Plugin ecosystem
- Good performance

**Best for:** Small to medium web applications, prototypes.

**Website:** https://perldancer.org/

### Database Tools

#### DBI

**The standard database interface.**

```perl
use DBI;

my $dbh = DBI->connect(
    "dbi:Pg:dbname=myapp",
    $username,
    $password,
    { RaiseError => 1, AutoCommit => 1 }
);

my $sth = $dbh->prepare("SELECT * FROM users WHERE id = ?");
$sth->execute($user_id);

while (my $row = $sth->fetchrow_hashref) {
    print $row->{name}, "\n";
}
```

**Best for:** Direct SQL control, simple database operations.

#### DBIx::Class

**Powerful ORM (Object-Relational Mapper).**

```perl
my $schema = MyApp::Schema->connect(@connect_info);

# Find user
my $user = $schema->resultset('User')->find($id);

# Create new record
my $new_user = $schema->resultset('User')->create({
    name => 'Alice',
    email => 'alice@example.com',
});

# Search with conditions
my @active_users = $schema->resultset('User')->search({
    status => 'active',
})->all;
```

**Features:**
- Object-oriented database access
- Relationship handling
- Migration tools
- Query building

**Best for:** Complex database schemas, relationships, migrations.

---

## Testing Tools

### Test::More

**Standard testing framework.**

```perl
use Test::More tests => 5;

ok($result, "Operation succeeded");
is($value, 42, "Value is 42");
isnt($value, 0, "Value is not zero");
like($string, qr/pattern/, "String matches pattern");
is_deeply(\@got, \@expected, "Arrays match");
```

### Test::Exception

**Test exception handling.**

```perl
use Test::More;
use Test::Exception;

dies_ok { divide(10, 0) } "Dividing by zero dies";

throws_ok { 
    divide(10, 0) 
} qr/Division by zero/, "Correct error message";

lives_ok { 
    divide(10, 2) 
} "Normal division succeeds";
```

### Test::MockModule

**Mock modules for testing.**

```perl
use Test::MockModule;

my $mock = Test::MockModule->new('MyModule');

$mock->mock('expensive_operation', sub {
    return 'mocked_result';
});

# Test code that calls expensive_operation()
# It will use mocked version instead
```

### Devel::Cover

**Test coverage analysis (covered in Section 12).**

```bash
cover -test
# Generates HTML coverage report
```

---

## Additional Learning Resources

### Online Tutorials

**Learn Perl.org**
- https://learn.perl.org/
- Official beginner resources
- Tutorials and documentation links

**Perl Maven**
- https://perlmaven.com/
- Extensive tutorial collection
- Video courses available
- Newsletter and blog

**Perl.com**
- https://www.perl.com/
- Articles and tutorials
- Best practices
- Community news

### Video Resources

**YouTube Channels:**
- **The Perl Conference** - Conference talks
- **Perl Maven** - Tutorial videos
- **Damian Conway** - Advanced talks

### Practice and Challenges

**Perl Weekly Challenge**
- https://perlweeklychallenge.org/
- Weekly coding challenges
- Solutions from community
- Great for learning

**Exercism Perl Track**
- https://exercism.org/tracks/perl5
- Practice exercises with mentorship
- Learn through solving problems

**Project Euler**
- https://projecteuler.net/
- Mathematical programming challenges
- Good for algorithm practice

### Perl for Specific Domains

**Bioinformatics:**
- BioPerl (https://bioperl.org/)
- Bioinformatics algorithms

**System Administration:**
- Perl for sysadmins resources
- Configuration management scripts

**Web Scraping:**
- Web::Scraper
- Mojo::UserAgent
- LWP::UserAgent

**Text Processing:**
- Natural Language Processing
- Regular expressions mastery

---

## Resources Checklist

### Getting Started (Week 1)

- [ ] Read `perldoc perlintro`
- [ ] Browse MetaCPAN for popular modules
- [ ] Install `cpanm` for easy module installation
- [ ] Join Reddit r/perl or PerlMonks
- [ ] Subscribe to Perl Weekly newsletter

### Building Foundation (Month 1)

- [ ] Read "Learning Perl" or complete online tutorial
- [ ] Complete Perl exercises on Exercism
- [ ] Read `perldoc perlsyn` and `perldoc perlfunc`
- [ ] Install and explore Mojolicious::Lite
- [ ] Write and publish a simple script

### Intermediate Development (Months 2-3)

- [ ] Read "Modern Perl" (free online)
- [ ] Study "Perl Best Practices" by Damian Conway
- [ ] Set up Perl::Critic and Perl::Tidy
- [ ] Write tests with Test::More
- [ ] Contribute to an open-source Perl project

### Advanced Mastery (Ongoing)

- [ ] Read "Programming Perl" for reference
- [ ] Study "Higher-Order Perl" for advanced techniques
- [ ] Attend The Perl Conference (virtual or in-person)
- [ ] Write and publish a CPAN module
- [ ] Mentor others in the community

### Daily Development

- [ ] Keep `perldoc` handy for quick reference
- [ ] Use MetaCPAN to find solutions before reinventing
- [ ] Follow Perl Weekly for latest news
- [ ] Participate in Perl Weekly Challenge
- [ ] Share knowledge through blog posts or talks

### Quick Reference Sites (Bookmark These)

- [ ] https://metacpan.org/ - Module search
- [ ] https://perldoc.perl.org/ - Documentation
- [ ] https://perlmonks.org/ - Community Q&A
- [ ] https://www.perl.com/ - Articles and news
- [ ] https://perlweekly.com/ - Weekly newsletter
- [ ] https://learn.perl.org/ - Official learning resources


---


[↑ Back to TOC](#table-of-contents)

