<div align="center">
  <h1>Formal Methods</h1>
  <p>
    <strong>It uses mathematical rigour to describe/specify systems before they get implemented</strong>
  </p>
</div>

Formal methods are techniques used by software engineers to design safety-critical systems and their components. In software engineering, they are techniques that involve mathematical expressions to model abstract representation of the system.

## K Framework

[kframework.org](https://kframework.org/)

### Install

See [runtimeverification/k/releases](https://github.com/runtimeverification/k/releases/tag/v5.3.156)

```shell
wget https://github.com/runtimeverification/k/releases/download/v5.3.156/kframework_5.3.156_amd64_focal.deb
sudo apt install ./kframework_5.3.156_amd64_focal.deb
```

See [editor Support](https://kframework.org/editor_support/)

### Use

#### Definition new language `lambda`

Create file definition syntax new language `lambda` and specification of behavior [`lambda.k`](k/lambda.k)

```k
require "substitution.md"

module LAMBDA-SYNTAX
  imports DOMAINS-SYNTAX
  imports KVAR-SYNTAX

  syntax Val ::= KVar
               | "lambda" KVar "." Exp [binder]
  syntax Exp ::= Val
               | Exp Exp [strict, left]
               | "(" Exp ")" [bracket]

  syntax Val ::= Int | Bool
  syntax Exp ::= "-" Int
               > Exp "*" Exp [strict, left]
               | Exp "/" Exp [strict]
               > Exp "+" Exp [strict, left]
               > Exp "<=" Exp [strict]

  syntax Exp ::= "if" Exp "then" Exp "else" Exp [strict(1)]

  syntax Exp ::= "let" KVar "=" Exp "in" Exp
  rule let X = E in E':Exp => (lambda X . E') E [macro]

  syntax Exp ::= "letrec" KVar KVar "=" Exp "in" Exp
               | "mu" KVar "." Exp [binder]
  rule letrec F:KVar X = E in E' => let F = mu F . lambda X . E in E' [macro]
endmodule

module LAMBDA
  imports LAMBDA-SYNTAX
  imports SUBSTITUTION
  imports DOMAINS

  syntax KResult ::= Val

  rule (lambda X:KVar . E:Exp) V:Val => E[V / X]

  rule - I => 0 -Int I
  rule I1 * I2 => I1 *Int I2
  rule I1 / I2 => I1 /Int I2
  rule I1 + I2 => I1 +Int I2
  rule I1 <= I2 => I1 <=Int I2

  rule if true  then E else _ => E
  rule if false then _ else E => E

  rule mu X . E => E[(mu X . E) / X]
endmodule
```

#### Compile

```shell
kompile --warnings none lambda.k
# or
make -C k compile
```

#### Create program on `lambda`

Create file [`fib.lambda`](k/fib.lambda)

```lambda
func fib x = if x <= 1 then 1 else (x * (fib (x + -1)))
in (fib 10)
```

#### Get `AST`

```shell
kast fib.lambda
# or
make -C k ast
```

#### Run

```shell
krun fib.lambda
# or
make -C k run
```
