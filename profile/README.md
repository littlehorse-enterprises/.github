<p align="center">
  <img alt="LH" src="../assets/logo-square.png" width="10%">
</p>

<a href="https://littlehorse.io/"><img alt="littlehorse.io" src="../assets/site-badge.svg"/></a>
<a href="https://littlehorse.io/docs/getting-started/quickstart"><img alt="littlehorse.io/learn" src="../assets/learn-badge.svg"/></a>
<a href="https://littlehorse.io/docs"><img alt="littlehorse.io/docs" src="../assets/docs-badge.svg"/></a>


[LittleHorse](https://littlehorse.io) is the pioneer of the [Business-as-Code](https://littlehorse.io/blog/business-as-code) approach to codifying and orchestrating processes across multiple systems. Built with love by developers, for developers!

Our Business-as-Code approach lets you write code in a language of your choice (Java, Python, Go, C#, Typescript) to specify processes which are durably orchestrated across your systems. We enable Business-as-Code primarily through the [LittleHorse Server](https://github.com/littlehorse-enterprises/littlehorse) which is our distributed workflow engine for Business-as-Code. Business-as-Code provides a higher level of abstraction for software engineers to work closer to the business process and (therefore) closer to _business value._


## What is Business-as-Code?

Business-as-Code is the practice of codifying and orchestrating processes across multiple SaaS systems, microservices, agents, and people. Business-as-Code explicitly specifies process logic at the business level from the top-down rather than letting processes emerge bottom-up via brittle point-to-point connections between systems. Developers use Business-as-Code to codify diverse processes ranging from back-office, manual workflows to SaaS integration workflows to real-time, event-driven microservice workflows.

<img alt="LH" src="https://littlehorse.io/img/dashboard-wfrun-running.png" width="100%">

:point_up: This picture shows a running instance (`WfRun`) for the process (`WfSpec`) defined by this code :point_down:

```java
public void quickstartWf(WorkflowThread wf) {
    WfRunVariable fullName = wf.declareStr("full-name").searchable().required();
    WfRunVariable email = wf.declareStr("email").searchable().required();

    // Social Security Numbers are sensitive, so we mask the variable with `.masked()`.
    WfRunVariable ssn = wf.declareInt("ssn").masked().required();

    WfRunVariable identityVerified = wf.declareBool("identity-verified").searchable();

    wf.execute(VERIFY_IDENTITY_TASK, fullName, email, ssn).withRetries(3);

    NodeOutput identityVerificationResult = wf.waitForEvent(IDENTITY_VERIFIED_EVENT)
            .timeout(60 * 5) // 5 minute timeout
            .withCorrelationId(email)
            .registeredAs(Boolean.class);

    wf.handleError(identityVerificationResult, LHErrorType.TIMEOUT, handler -> {
        handler.execute(NOTIFY_CUSTOMER_NOT_VERIFIED_TASK, fullName, email);
        handler.fail("customer-not-verified", "Unable to verify customer identity in time.");
    });

    identityVerified.assign(identityVerificationResult);

    wf.doIf(identityVerified.isEqualTo(true), ifBody -> {
        ifBody.execute(NOTIFY_CUSTOMER_VERIFIED_TASK, fullName, email);
    })
    .doElse(elseBody -> {
        elseBody.execute(NOTIFY_CUSTOMER_NOT_VERIFIED_TASK, fullName, email);
    });
}
```

As you can see, the code above closely mirrors our example KYC business process. LittleHorse handles retries, timeouts, and orchestration across services for you, allowing your `WfSpec` to focus just on what matters to the business. Task workers handle integrations with external systems and databases.
