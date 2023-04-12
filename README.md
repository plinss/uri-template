# uri-template

An implementation of RFC 6570 URI Templates.

This packages implements URI Template expansion in strict adherence to RFC 6570,
but adds a few extensions.

## RFC 6570 Extensions

### Non-string Values

RFC 6570 is silent regarding variable values that are not strings, lists, associative arrays, or null.

This package handles value types as follows:

  * Values that are instances of `str` are treated as strings.
  * Values implementing `collections.abc.Sequence` are treated as lists.
  * Values implementing `collections.abc.Mapping` are treated as associative arrays.
  * `None` values are treated as null.
  * Boolean values are converted to the lower case strings 'true' and 'false'.
  * All other values will be converted to strings using the Python `str()` function.

### Nested Structures

This package handles variable values with nested structure,
for example, lists containing other lists or associative arrays,
or associative arrays containing lists or other associative arrays.

Nested values for variables that do not use the array modifier ('[]') are treated as follows:

  * Lists containing lists are flattened into a single list.
  * Lists containing associative arrays are treated as a single combined associative array.
  * Associative arrays represent nested data using dot notation (".") for the variable names.

Nested values for variables that use the array modifier extend the variable name with 
the value's index or key written as an array subscript, e.g. "foo[0]" or "foo[bar]".

### Default Values

This package allows default string values for variables per early drafts of RFC 6570.
e.g. "{foo=bar}" will expand to "bar" if a value for `foo` is not given.

List and associtative array default values are not supported at this time.

### Partial expansion

This package allows partial expansion of URI Templates.

In a partial expansion, missing values preseve their expansion in the resultant output.
e.g. a partial expansion of "{one}/{two}" with a value for `one` of "foo" and `two` missing will result in:
"foo/{two}".

In order to allow partial expansions to preserve value joiners with expanded output,
expansions accept an optional "trailing joiner" of ",", ".", "/", ";", or "&",
if this joiner is present after all variables, 
it will be appended to the output of the expansion and will suppress the output prefix.
e.g.: "{#one,two}" with a missing value for `one` and a value of "bar" for `two`, 
will partially expand to: "#{#one,}bar", which when provided with a value of "foo" for `one` 
will expand to "#foo,bar"

Some partial expansions that have some output, but have missing values, 
will convert the remaining variables to a different type of expansion so that 
further expansions will produce the same output as if all values were originally present.

   * Partial Simple String Expansions will convert to Comma Expansions.
   * Partial Reserved Expansions Partial Fragment Expansions will convert to Reserved Comma Expansions.
   * Partial Form-Style Query Expansions will convert to Form-Style Query Continuations.

In order to preserve the resultant value of templates that are paritally expanded, 
the following additional Expression Expansions are supported:

#### Comma Expansion: {,var}

Similar to Label Expansion with Dot-Prefix, 
Comma Expansion prefixes the expansion output with a single comma ",".

#### Reserved Comma Expansion: {,+var}

Similar to Comma Expansion, 
Reserved Comma Expansion prefixes the expansion output with a single comma ",",
but otherwise performs a Reserved Expansion ({+var}).

## API 

The package provides three functions:

#### uri_template.expand(template: str, **kwargs) -> (str | None): ...

Expand the given template, skipping missing values per RFC 6570.

Returns `None` if the template is invalid or expansion fails.


#### uri_template.partial(template: str, **kwargs) -> (str | None): ...

Partially expand the given template, 
replacing missing variables with further expansions.

Returns `None` if the template is invalid or expansion fails.


#### uri_template.validate(template: str) -> bool: ...

Return `True` if the template is valid.

---

And the following classes:

### uri_template.URITemplate

#### URITemplate(template: str)

Construct a URITemplate for a given template string.

Raises `ExpansionInvalid`, `ExpansionReserved`, or `VariableInvalid` if the template is invalid or unsupported.

#### URITemplate.variables: Iterable[Variable]

All variables present in the template.
Duplicates are returned once, order is preserved.

#### URITemplate.variable_names: Iterable[str]

The names of all variables present in the template.
Duplicates are returned once, order is preserved.

#### URITemplate.expanded: bool

Determine if template is fully expanded.

#### URITemplate.expand(**kwargs) -> str

Returns the result of the expansion, skips missing variables.

Raises `ExpansionFailed` if the expansion fails due to a composite value being passed to a variable with a prefix modifier.

#### URITemplate.partial(**kwargs) -> URITemplate

Expand the template, replacing missing variables with further expansions.

Raises `ExpansionFailed` if the expansion fails due to a composite value being passed to a variable with a prefix modifier.

#### URITemplate.__str__() -> str

Convert the URITemplate object back into its original string form.

---

### uri_template.Variable

#### Variable(var_spec: str)

Construct a Variable.

#### Variable.name: str

The name of the variable

#### Variable.max_length: int

The speicified max length, or `0`.

#### Variable.explode: bool

Explode modifier is present.

#### Variable.array: bool

Array modifier is present.

#### Variable.default: (str | None)

Specified default value, or `None`.

#### Variable.__str__() -> str

Convert the variable back to its original string form.

---

And the following exceptions:

#### uri_template.ExpansionInvalid

Expansion specification is invalid. 

Raised by URITemplate constructor.

#### uri_template.ExpansionReserved

Expansion contains a reserved operator.

Raised by URITemplate constructor.

#### uri_template.VariableInvalid

Variable specification is invalid.

Raised by URITemplate constructor.

#### uri_template.ExpansionFailed

Expansion failed, currently only possible when a composite value is passed to a variable with a prefix modifier.

Raised by URITemplate.expand() or URITemplate.partial() methods.


## Installation

Install with pip:

    pip install uri-template