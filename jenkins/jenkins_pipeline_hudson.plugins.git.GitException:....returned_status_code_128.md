```bash
Started by user ghostwritten
Loading library shared-library@main
Attempting to resolve main from remote references...
 > git --version # timeout=10
 > git --version # 'git version 2.38.1'
using GIT_ASKPASS to set credentials github-token
 > git ls-remote -h -- https://github.com/Ghostwritten/github-api-global-lib.git # timeout=10
ERROR: Checkout failed
hudson.plugins.git.GitException: Command "git ls-remote -h -- https://github.com/Ghostwritten/github-api-global-lib.git" returned status code 128:
stdout: 
stderr: fatal: unable to access 'https://github.com/Ghostwritten/github-api-global-lib.git/': Empty reply from server

	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandIn(CliGitAPIImpl.java:2734)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandWithCredentials(CliGitAPIImpl.java:2111)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandWithCredentials(CliGitAPIImpl.java:2009)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.launchCommandWithCredentials(CliGitAPIImpl.java:2000)
	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl.getRemoteReferences(CliGitAPIImpl.java:3671)
	at jenkins.plugins.git.AbstractGitSCMSource.retrieve(AbstractGitSCMSource.java:862)
	at jenkins.scm.api.SCMSource.fetch(SCMSource.java:636)
	at org.jenkinsci.plugins.workflow.libs.SCMSourceRetriever.lambda$retrieve$0(SCMSourceRetriever.java:133)
	at org.jenkinsci.plugins.workflow.libs.SCMSourceRetriever.retrySCMOperation(SCMSourceRetriever.java:148)
	at org.jenkinsci.plugins.workflow.libs.SCMSourceRetriever.retrieve(SCMSourceRetriever.java:133)
	at org.jenkinsci.plugins.workflow.libs.LibraryAdder.retrieve(LibraryAdder.java:260)
	at org.jenkinsci.plugins.workflow.libs.LibraryAdder.add(LibraryAdder.java:150)
	at org.jenkinsci.plugins.workflow.libs.LibraryDecorator$1.call(LibraryDecorator.java:125)
	at org.codehaus.groovy.control.CompilationUnit.applyToPrimaryClassNodes(CompilationUnit.java:1087)
	at org.codehaus.groovy.control.CompilationUnit.doPhaseOperation(CompilationUnit.java:624)
	at org.codehaus.groovy.control.CompilationUnit.processPhaseOperations(CompilationUnit.java:602)
	at org.codehaus.groovy.control.CompilationUnit.compile(CompilationUnit.java:579)
	at groovy.lang.GroovyClassLoader.doParseClass(GroovyClassLoader.java:323)
	at groovy.lang.GroovyClassLoader.parseClass(GroovyClassLoader.java:293)
	at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.GroovySandbox$Scope.parse(GroovySandbox.java:163)
	at org.jenkinsci.plugins.workflow.cps.CpsGroovyShell.doParse(CpsGroovyShell.java:142)
	at org.jenkinsci.plugins.workflow.cps.CpsGroovyShell.reparse(CpsGroovyShell.java:127)
	at org.jenkinsci.plugins.workflow.cps.CpsFlowExecution.parseScript(CpsFlowExecution.java:563)
	at org.jenkinsci.plugins.workflow.cps.CpsFlowExecution.start(CpsFlowExecution.java:515)
	at org.jenkinsci.plugins.workflow.job.WorkflowRun.run(WorkflowRun.java:336)
	at hudson.model.ResourceController.execute(ResourceController.java:107)
	at hudson.model.Executor.run(Executor.java:449)
ERROR: Maximum checkout retry attempts reached, aborting
org.codehaus.groovy.control.MultipleCompilationErrorsException: startup failed:
WorkflowScript: Loading libraries failed
```
解决方法：

```bash
ssh-keygen -t rsa -C "<github绑定的邮箱地址>"
```
复制  `id_rsa.pub` 至 github 设置 `SSH keys`保存。

测试

```bash
$ git ls-remote -h -- https://github.com/Ghostwritten/github-api-global-lib.git
bfe1e149a04416cf4bcfb58dfe719695b17936d0        refs/heads/main
```
成功
