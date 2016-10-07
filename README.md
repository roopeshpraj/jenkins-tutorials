# jenkins-tutorials
https://wiki.jenkins-ci.org/display/JENKINS/Job+DSL+Plugin
https://github.com/jenkinsci/job-dsl-plugin/wiki
https://jenkinsci.github.io/job-dsl-plugin/

Basic Use
Write a seed job
This contains the config for all of the jobs it creates, and is linked to these jobs.
Every time the seed job is run, the actions by (default) are:
Create any new jobs specified
Update all existing jobs linked to the previous build of the DSL job to match the new DSL
This includes removing any customisations to the jobs made by a user
Optional: Delete/Disable/Ignore jobs no longer specified by the template. (N.B. If you run a seed job with "Action for removed jobs:" set to "Ignore", any "unreferenced" jobs will be unlinked from the Job DSL job and won't ever be touched by it again. If you are working with lots of jobs, it is better to delete them then leave them floating around).
Converting projects from XML templates to Job DSL
Use this reference https://jenkinsci.github.io/job-dsl-plugin. Find commands under same groups as the tags in the config.xml. For example, the workspace cleanup comes in the buildWrappers tag:
<buildWrappers>
<hudson.plugins.ws__cleanup.PreBuildCleanup plugin="ws-cleanup@0.26">
<deleteDirs>false</deleteDirs>
<cleanupParameter/>
<externalDelete/>
</hudson.plugins.ws__cleanup.PreBuildCleanup>
So we look in the wrappers section of the DSL reference.
https://jenkinsci.github.io/job-dsl-plugin/#path/job-wrappers-preBuildCleanup
Plugins that haven't exposed Job DSL Syntax
See https://github.com/jenkinsci/job-dsl-plugin/wiki/The-Configure-Block
e.g.
 configure { config ->
      config / 'properties' / 'hudson.plugins.copyartifact.CopyArtifactPermissionProperty' / 'projectNameList' {
            'string'('android-deployer')
        }
      }
Becomes:
<hudson.plugins.copyartifact.CopyArtifactPermissionProperty>
<projectNameList>
<string>android-deployer</string>
</projectNameList>
</hudson.plugins.copyartifact.CopyArtifactPermissionProperty>
The Jenkins job configuration page provides validation of values. Make use of it!
For example, when setting up a "Build periodically" trigger
If we type in this:
H H(22) * * *
We get an error, so we know to correct it - this works:
H H(22-23) * * *
So in Job DSL world the trigger command we want is:
triggers {
   cron('H H(22-23) * * *')
}
And we can be confident that the value we've fed into the cron() command is valid.
Detailed Information
Does running a seed job periodically bloat the Job Config History?
Yes. If the item appears to the Job DSL plugin to have been changed (which it nearly always will; see issue below "This item has been manually changed"), the config will be replaced again, and history added to. The history of the seed job is more important with Job DSL, and access to Job Configuration is strictly limited, so the Job Config History can be removed.
Can we keep the build history if we overwrite an existing job using Job DSL?
Yes. It is kept.
Need to change job names?
See https://jenkinsci.github.io/job-dsl-plugin/#path/javaposse.jobdsl.dsl.DslFactory.mavenJob-previousNames
Can you run a SeedJob while a build is running?
Yes. Tested with Pipelines and FreeStyle jobs. Config is updated for next build only, not a currently building build.
Troubleshooting
This type of error implies a syntax error in the Job DSL you have written:
Processing provided DSL script
ERROR: (script, line 26) No signature of method: javaposse.jobdsl.dsl.jobs.FreeStyleJob.preBuildCleanup() is applicable for argument types: (script$_run_closure1_closure2_closure5) values: [script$_run_closure1_closure2_closure5@4e88b71e]
"This item has been changed manually…" message:
When you navigate to a job, Jenkins seems to check it is happy with the XML. Sometimes Jenkins then regenerates it, saving it as if your user had changed it, and the change could be just new line breaks or ordering of the XML. This seems to be a bug in the Job DSL plugin. However it will also remove any bad config, e.g. something that Jenkins says is not valid, so best check the config page to see the job is how you want it.
Gotchas
"Unreferenced" jobs still appear linked to from the Seed Job (until deleted)
