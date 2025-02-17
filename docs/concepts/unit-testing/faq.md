# Frequently Asked Questions

This section covers questions that aren't answered in the Spock or Jenkins-Spock documentation.

**Q: What's the difference between explicitlyMockPipelineStep and explicitlyMockPipelineVariable?**
**A: Practically speaking, the difference is you can omit ".call" for explicitlyMockPipelineStep() when you use getPipelineMock()**

In the example on the [Writing Tests](./writing-tests.md) page, ``explicitlyMockPipelineStep()`` is used to mock `error`.
In this case, `1 * getPipelineMock("error")` can be used to see if the `error` pipeline step is run.
If `explicitlyMockPipelineVariable()` had been used instead, `1 * getPipelineMock("error.call")` could be used to check for the `error` pipeline step.

There may be some additional differences as well, so try to use what makes the most sense.

**Q: What if the exact parameters are unknown?**
**A: There are ways to match parameters to regex expressions, as well as test parameters individually**

The standard format for interaction-based tests is:

```groovy
<count> * getPipelineMock(<method>)(<parameter(s)>)
```

While you can put the exact parameter value in the second parentheses, you can also run arbitrary groovy code inside curly brackets.
If it's a "match" depends on if that code returns `true` or `false`.
A good example is in [PenetrationTestSpec.groovy](https://github.com/boozallen/sdp-libraries/blob/main/libraries/owasp_zap/test/PenetrationTestSpec.groovy#L33). Use `it` to get the value of the parameter:
`1 * getPipelineMock("sh")({it =~ / (zap-cli open-url) Kirk (.+)/})`.

**Q: Is interaction-based testing required?**
**A: No, but you can't get variables the same way as traditional Spock tests**

This is because the script gets run within the `loadPipelineScriptForTest` object.
You can only access the limited set of variables stored in the binding.
It makes more sense to see how variables are being used in pipeline steps,
and make sure those pipeline steps use the correct value for those variables.

Similarly, if you need to control how a variable is set,
you need to stub whatever method or pipeline step sets the initial value for that variable.

As an example, in [PenetrationTestSpec.groovy](https://github.com/boozallen/sdp-libraries/blob/main/libraries/owasp_zap/test/PenetrationTestSpec.groovy),
the `target` variable in [penetration_test.groovy](https://github.com/boozallen/sdp-libraries/blob/main/libraries/owasp_zap/steps/penetration_test.groovy) is tested by checking the parameters to an `sh` step.

**Q: What troubleshooting steps are required for "can't run method foo() on null" errors?"**
**A: You need to find a way to stub the method that sets the value for the object that calls foo()**

Check out an example in [GetImagesToBuildSpec.groovy](https://github.com/boozallen/sdp-libraries/blob/main/libraries/docker/test/GetImagesToBuildSpec.groovy).
