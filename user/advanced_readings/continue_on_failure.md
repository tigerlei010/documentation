# Continue on Failure

The default behaviour in Gauge is to break execution on the first failure in a [step](../gauge_terminologies/steps.md). So, if the first step in a [scenario](../gauge_terminologies/scenarios.md) fails, the subsequent steps are skipped. While this works for a majority of use cases, there are times when you need to execute all steps in a scenario irrespective of whether the previous steps have failed or not.

To address that requirement, Gauge provides a way for language runners to mark steps as recoverable, depending on whether the step implementation asks for it explicitly. Each language runner uses different syntax, depending on the language idioms, to allow a step implementation to be marked to continue on failure.

## Usage

{% codetabs name="Java", type="java" -%}
// The `@ContinueOnFailure` annotation tells Gauge to continue executing other
// steps even if the current step fails.

public class StepImplementation {

    @ContinueOnFailure
    @Step("Say <greeting> to <product name>")
    public void helloWorld(String greeting, String name) {
        // If there is an error here, Gauge will still execute next steps
    }
}
{%- language name="C#", type="csharp" -%}
// The `[ContinueOnFailure]` attribute tells Gauge to continue executing other
// steps even if the current step fails.

public class StepImplementation {

    [ContinueOnFailure]
    [Step("Say <greeting> to <product name>")]
    public void HelloWorld(string greeting, string name) {
        // If there is an error here, Gauge will still execute next steps
    }
}
{%- language name="Ruby", type="ruby" -%}
# The `:continue_on_failure => true` keyword argument tells Gauge to continue executing
# other steps even if the current step fails.

step 'Say <greeting> to <product_name>', :continue_on_failure => true do |greeting, name|
    # If there is an error here, Gauge will still execute next steps
end

{%- endcodetabs %}

Continue on Failure can take an optional parameter to specify the list of error classes on which it would continue to execute further steps in case of failure. This is currently supported only with Java runner.

{% codetabs name="Java", type="java" -%}
@ContinueOnFailure({AssertionError.class, CustomError.class})
@Step("hello")
public void sayHello() {
    // code here
}
@ContinueOnFailure(AssertionError.class)
@Step("hello")
public void sayHello() {
    // code here
}
@ContinueOnFailure
@Step("hello")
public void sayHello() {
    // code here
}
{%- endcodetabs %}

In case no parameters are passed to `@ContinueOnFailure`, on any type of error it continues with execution of further steps by default.

This can be used to control on what type of errors the execution should continue, instead of just continuing on every type of error. For instance, on a `RuntimeException` it's ideally not expected to continue further. Whereas if it's an assertion error, it might be fine to continue execution.


## Caveats

- Continue on failure comes into play at post execution, i.e. after the step method is executed. If there is a failure in executing the step, ex. parameter count/type mismatch, Gauge will not honour the `ContinueOnFailure` flag.
- Continue on failure does not apply to [hooks](../language_features/execution_hooks.md). Hooks always fail on first error.
- Step implementations are still non-recoverable by default and Gauge does not execute subsequent steps upon failure. To make a step implementation continue on failure, it needs to be explicitly marked in the test code.
- There is no way to globally mark a test run to treat all steps to continue on failure. Each step implementation has to be marked explicitly.
- If an implementation uses step aliases, marking that implementation to continue on failure will also make all the aliases to continue on failure. So, if a step alias is supposed to break on failure and another step alias is supposed to continue on failure, they need to be extracted to two different step implementations.
