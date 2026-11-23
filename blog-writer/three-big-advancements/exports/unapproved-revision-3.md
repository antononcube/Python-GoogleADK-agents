# Three Recent Additions to Raku Numerics

Anton Antonov  
November 2026

## Introduction

This post surveys three areas of numerical and statistical computing in Raku:

1. numerical integration with [`Math::NIntegrate`](https://github.com/antononcube/Raku-Math-NIntegrate);
2. mathematical optimization with [`Math::Optimization`](https://github.com/antononcube/Raku-Math-Optimization) and [`COIN-OR`](https://github.com/antononcube/Raku-COIN-OR);
3. quantile regression and regression workflows with [`Math::QuantileRegression`](https://github.com/antononcube/Raku-Math-QuantileRegression) and [`Math::Regressionizer`](https://github.com/antononcube/Raku-Math-Regressionizer).

The areas overlap. In particular, quantile regression is commonly formulated as an optimization problem, and the higher-level workflow tools build on numerical and statistical routines supplied by other packages.

This is an overview, not a complete tutorial. The package repositories contain additional examples and implementation details.

## Installation and reproducibility

The APIs of Raku ecosystem packages can change between revisions. The examples in this article are based on package documentation retrieved in November 2026. Before using them in a project, test them against the exact Raku distribution and package revisions used by that project.

A typical installation begins with:

```sh
zef install Math::NIntegrate
zef install DSL::English::IntegrationWorkflows

zef install COIN-OR
zef install Math::Optimization

zef install Math::QuantileRegression
zef install Math::Regressionizer
zef install DSL::English::QuantileRegressionWorkflows
zef install Data::Generators
zef install JavaScript::Google::Charts
```

Some packages have dependencies that `zef` installs automatically; the explicit list above makes the examples easier to identify and reproduce. In particular, `Math::Regressionizer` uses `JavaScript::Google::Charts` for plotting.

`COIN-OR` can require native solver libraries, headers, and platform-specific configuration. Consult the `COIN-OR` package documentation and the documentation of the selected solver backend before installing it on a new platform.

For reproducible work, record:

- the Raku compiler version;
- the operating system and native solver-library versions;
- the resolved versions or commit hashes of all Raku packages;
- the package-manager output produced during installation.

The repository links in this article are useful entry points, but a production workflow should use release archives, immutable commit hashes, or another version-pinning mechanism.

---

## Numerical integration

[`Math::NIntegrate`](https://github.com/antononcube/Raku-Math-NIntegrate) is a Raku package for numerical integration. Its `nintegrate` framework resembles the design of the Wolfram Language function `NIntegrate`, while using Raku classes and extension mechanisms.

The documented one-dimensional integration rules include:

- Clenshaw–Curtis;
- Gauss–Kronrod;
- Newton–Cotes;
- trapezoidal rules.

The documented multidimensional rule families include:

- Cartesian rules, assembled from one-dimensional rules;
- multidimensional rules;
- double-exponential rules;
- Monte Carlo rules;
- quasi-Monte Carlo rules;
- user-defined rules.

The package also documents the following integration strategies:

- global adaptive integration;
- local adaptive integration;
- double-exponential integration;
- Monte Carlo integration;
- quasi-Monte Carlo integration;
- adaptive Monte Carlo integration;
- adaptive quasi-Monte Carlo integration;
- user-defined strategies.

A unit-cube-rescaling strategy decorator is also documented. Global adaptive integration is the documented default strategy. When a one-dimensional rule is used for a multidimensional integral, the framework can automatically convert it into a Cartesian rule.

### Architecture and extension points

`Math::NIntegrate` separates rules from strategies:

- A **rule** approximates an integral over a particular region.
- A **strategy** controls the overall integration process, such as adaptive subdivision or stochastic sampling.
- A **strategy decorator** contains another strategy.
- A strategy that is not a decorator contains a rule.
- A **composite rule**, such as a Cartesian rule, contains one or more component rules.

The package documentation describes an object-oriented design that employs Strategy, Composite, Decorator, Template Method, Builder, Interpreter, and Abstract Factory patterns. This structure provides extension points for user-defined rules and strategies.

### Natural-language integration specifications

[`DSL::English::IntegrationWorkflows`](https://github.com/antononcube/Raku-DSL-English-IntegrationWorkflows) translates selected English-like integration descriptions into Raku source code.

For example:

```raku
use DSL::English::IntegrationWorkflows;

my $command =
    'numerically integrate 1/x^2 over the range [1,2] '
    ~ 'with local adaptation and precision goal 5';

my $source = ToIntegrationWorkflowCode(
    $command,
    'Raku::Ecosystem'
);

say $source;
```

The documented translation corresponds to an expression for

\[
\int_1^2 \frac{1}{x^2}\,dx = \frac{1}{2}.
\]

The important distinction is that `ToIntegrationWorkflowCode` generates Raku code; it does not itself perform the integration. Generated code should be reviewed before execution.

Do not directly evaluate generated code derived from untrusted users, external data, or an LLM response. If an application must accept untrusted natural-language specifications, it should use a constrained grammar, a whitelist of supported functions and options, and explicit validation before evaluation.

### AI-assisted development

The following notes summarize the author’s development account. The core object-oriented relationships among regions, variable transformations, rules, and strategies were implemented directly. AI assistance was used selectively for drafting some integration rules, translating legacy numerical code, and generating candidate tests.

This illustrates an important limitation of AI-assisted numerical programming: generated code is only a starting point. Integration rules must be validated against known integrals, convergence behavior, and, where possible, independent implementations. Translations of multidimensional numerical code can require substantial review and correction.

---

## Mathematical optimization

[`Math::Optimization`](https://github.com/antononcube/Raku-Math-Optimization) provides Raku interfaces for mathematical-optimization tasks. Its documentation states that the [`COIN-OR`](https://github.com/antononcube/Raku-COIN-OR) Raku package must be installed.

Consider the linear-programming problem

\[
\begin{aligned}
\text{minimize}\quad & x+y \\
\text{subject to}\quad & x+2y \geq 3,\\
& -1 \leq x \leq 2,\\
& -1 \leq y \leq 1.
\end{aligned}
\]

The documented call is:

```raku
use Math::Optimization;

linear-programming(
    [1, 1],
    [(1, 2),],
    [3,],
    [[-1, 2], [-1, 1]]
)
```

For this example, the arguments represent:

1. objective coefficients, `[1, 1]`, for minimizing \(x+y\);
2. rows of the constraint matrix, here `(1, 2)`;
3. right-hand-side values, here `[3]`;
4. variable bounds, specified as lower–upper pairs.

The documented constraint orientation is

\[
A x \geq b.
\]

Therefore, the row `(1, 2)` and right-hand side `3` represent \(x + 2y \geq 3\). The documented result is:

```text
[1 1]
```

At \((x,y)=(1,1)\), the objective value is \(2\), and the constraint is satisfied at equality.

The example establishes the documented solution for this problem, but it should not be treated as a complete specification of return types, solver-status reporting, infeasibility handling, unboundedness handling, or backend errors. Applications should verify those details against their pinned package and solver versions.

Additional linear-programming examples are available in the package documentation, including [`LinearProgramming.md`](https://github.com/antononcube/Raku-Math-Optimization/blob/main/docs/LinearProgramming.md).

### AI-assisted development

According to the author’s development account, AI assistance was used while drafting parts of the `COIN-OR`, `Gurobi`, and `Math::Optimization` interfaces. A complete specification of `linear-programming` was prepared before generating the higher-level interface.

Regardless of how a solver interface is produced, it requires careful verification. A binding can be syntactically correct while reversing objective direction, misreading constraint orientation, mishandling bounds, or failing to report solver status correctly.

---

## Quantile regression and regression workflows

[`Math::QuantileRegression`](https://github.com/antononcube/Raku-Math-QuantileRegression) provides basic quantile-regression routines. Whereas least-squares regression estimates a conditional mean, quantile regression estimates conditional quantiles of a response variable.

For a probability \(p\), a fitted quantile-regression curve estimates the conditional \(p\)-quantile of the response. This can be useful when variability, asymmetry, or tail behavior changes with predictors. Roger Koenker’s *Quantile Regression* is a standard reference on the subject.

Quantile regression is commonly formulated as an optimization problem. The package documentation requires `COIN-OR`; users should verify the implementation dependencies and solver behavior for the selected package revision.

[`Math::Regressionizer`](https://github.com/antononcube/Raku-Math-Regressionizer) provides a class for pipeline-like specification and execution of regression workflows. Its documented features include:

- data summaries and rescaling;
- regression computation;
- outlier detection;
- conditional-distribution-function reconstruction;
- plotting of data, fits, residuals, outliers, and distribution functions.

The package documentation refers to the distribution-function feature as “conditional-CDF reconstruction.” Such reconstructions should be interpreted as model-based estimates, not as automatically validated probability models.

The package accepts several data representations, including datasets, numerical arrays, lists of numbers, and lists of numeric pairs. Its documented regression methods include:

- `quantile-regression`;
- `quantile-regression-fit`;
- `least-squares-fit`.

### Example: noisy sinusoidal data

The following code constructs observations from a sine curve with normally distributed noise:

```raku
use Data::Generators;

srand(0);

my @x = 0, 0 + 1/300 ... 2;

my @y =
    @x.map({ sin(2 * pi * $_) })
    <<+>>
    random-variate(
        NormalDistribution.new(0, 0.4),
        @x.elems
    );

my @data = @x Z @y;
```

The `Z` operator combines corresponding elements of `@x` and `@y`, producing pairs of the form `(x, y)`.

The following documented-style workflow fits regression quantiles at probabilities `0.2`, `0.5`, and `0.8`:

```raku
use Math::Regressionizer;

my $fit =
    Math::Regressionizer.new(@data)
    .quantile-regression(
        knots => 8,
        probs => [0.2, 0.5, 0.8]
    );
```

The package documentation describes `quantile-regression` as using a B-spline function basis. The relevant options include `:knots` and `:order`; the documented default interpolation order is `3`. Users should consult the documentation for their selected revision to determine the accepted form of `knots` and the exact relationship between the package’s interpolation order and spline degree.

The package documentation demonstrates plotting from the workflow object:

```raku
$fit.plot(
    title    => 'B-spline quantile-regression fit',
    template => 'dark',
    format   => 'html'
);
```

Plotting uses `JavaScript::Google::Charts`. The documentation demonstrates display in a Jupyter notebook with a Raku kernel. The representation and display behavior of HTML output can depend on the selected package revision and execution environment.

### Interpretation cautions

The package documents outlier detection and conditional-CDF reconstruction as workflow features. Their interpretation remains model-dependent:

- An outlier is defined relative to the fitted model and the selected workflow settings.
- Separately fitted quantile curves can cross; a reconstructed conditional distribution function may therefore require monotonicity checks or post-processing.
- Quantile-regression results should be assessed with domain knowledge and, where appropriate, compared with established statistical implementations.

### Natural-language workflow specifications

[`DSL::English::QuantileRegressionWorkflows`](https://github.com/antononcube/Raku-DSL-English-QuantileRegressionWorkflows) supports natural-language specifications of quantile-regression workflows.

As with integration-workflow generation, natural-language translation is a code-generation boundary. Commands generated from user-provided text or LLM output should be reviewed and validated before execution.

### AI-assisted development

According to the author’s development account, AI assistance was used while translating selected Wolfram Language fragments and drafting early versions of `Math::QuantileRegression`. The initial work on `Math::Regressionizer` also drew on the Python package [`Regressionizer`](https://pypi.org/project/Regressionizer/).

AI assistance can be useful for translation, boilerplate, test candidates, and documentation examples. It does not replace statistical validation. Regression implementations should be checked with reproducible synthetic data, known results, and comparisons with established tools where feasible.

---

## Conclusion

The packages discussed here extend Raku’s numerical-computing ecosystem in several directions:

- `Math::NIntegrate` provides an extensible framework for one- and multidimensional numerical integration.
- `Math::Optimization` provides documented interfaces for optimization tasks, including linear programming.
- `Math::QuantileRegression` and `Math::Regressionizer` provide quantile-regression routines and workflow-oriented regression tools.

The accompanying natural-language workflow packages can make these facilities easier to explore. However, generated source code remains source code: it should be inspected, version-pinned, and executed only within an appropriate trust boundary.

---

## Package resources and documentation basis

The following repositories are the primary package resources discussed in this article. The article was reviewed against their public README documentation retrieved in November 2026. Because repository default branches can change, readers should record an immutable commit hash or release identifier before relying on a particular API behavior.

- Anton Antonov, [`Math::NIntegrate`](https://github.com/antononcube/Raku-Math-NIntegrate), Raku package, 2026.
- Anton Antonov, [`DSL::English::IntegrationWorkflows`](https://github.com/antononcube/Raku-DSL-English-IntegrationWorkflows), Raku package, 2026.
- Anton Antonov, [`COIN-OR`](https://github.com/antononcube/Raku-COIN-OR), Raku package, 2026.
- Anton Antonov, [`Math::Optimization`](https://github.com/antononcube/Raku-Math-Optimization), Raku package, 2026.
- Anton Antonov, [`Math::QuantileRegression`](https://github.com/antononcube/Raku-Math-QuantileRegression), Raku package, 2026.
- Anton Antonov, [`Math::Regressionizer`](https://github.com/antononcube/Raku-Math-Regressionizer), Raku package, 2026.
- Anton Antonov, [`DSL::English::QuantileRegressionWorkflows`](https://github.com/antononcube/Raku-DSL-English-QuantileRegressionWorkflows), Raku package, 2022–2026.
- Roger Koenker, *Quantile Regression*, Cambridge University Press, 2005.