// variable for use in build steps where multiple projects are built in series
def slnName

/* Returns the list of recipients for build emails
 * Accessed in pipeline as: "$emailRecipients"
 */
def getEmailRecipients() {
    return 'prajput@fhlbi.com,softwaredeliveryprojectteam@fhlbi.com'
}

/* Returns the name of the project in SCM.
 * Accessed in pipeline as: "$srcProjectName"
 *
 * NOTE: This name should typically match the $PROJECT_NAME
 */
def getSrcProjectName() {
    return 'FHLBI.CreditSuite.Api'
}

/* Returns the prefix to use for source tagging
 * Accessed in pipeline as: "$tagPrefix"
 */
def getTagPrefix() {
    return 'FCSA'
}

/* Returns the name of the branch used for production releases
 * This branch is a protected branch that will be pushed to when a new release
 * is needed for deployment on [PROD] servers
 * - [QA] releases may also come from this branch when the project does not
 *   use a dedicated project-specific release branch.
 *   In those cases set the name of the [project release] branch to be the same
 *   as the [production release] branch.
 * - DO NOT SET THIS TO 'master'. BUILDS DO NOT OCCUR FROM THE [master] BRANCH
 *
 * BEHAVIORS:
 * - Builds from this branch are tagged
 * - Builds from this branch are scanned by Checkmarx
 * - Builds from this branch are pushed to Nexus (nuget-hosted)
 *
 * Accessed in pipeline as: "$productionReleaseBranch"
 */
def getProductionReleaseBranch() {
    return 'release'
}

/* Returns the name of the production integration branch
 * The Production integration branch is a branch used to create releases for
 * deployment on [SBX/DEV] that upgrade an existing installation.
 *
 * BEHAVIORS:
 * - Builds from this branch are pushed to Nexus (nuget-snapshots)
 *
 * Accessed in pipeline as: "$productionIntegrationBranch"
 */
def getProductionIntegrationBranch() {
    return 'integration'
}

///////////////////////////////////////////////////////////////////////////////
//
// THE FOLLOWING VALUES SHOULD BE UPDATED WHEN A BRANCH IS CREATED AT THE START
// OF A PROJECT TO ADD FEATURES OR WHEN BRANCHING FOR A MAINTENANCE RELEASE
//
// ALSO:
// When a NEW project is initiated the release_num (or the major_version_num)
// found in the parameters block should also be updated.
//
///////////////////////////////////////////////////////////////////////////////

/* Returns the name of the project-specific release branch
 * The Project release branch is a protected branch that will be pushed to when
 * a new release is needed for deployment on [QA] servers.
 * - MAY BE THE SAME NAME AS THE [production release] BRANCH.
 *   Projects that do not conflict with ongoing PAS team support releases may
 *   be using the [production release] branch as the [project release] branch.
 *   In those cases set the name of the [project release] branch to be the same
 *   as the [production release] branch.
 *   (example: IIS-CollateralUpload uses the [production release] branch ONLY
 *    and uses the value 'release' for this property)
 *
 * BEHAVIORS:
 * - Builds from this branch are tagged
 * - Builds from this branch are scanned by Checkmarx
 * - Builds from this branch are pushed to Nexus (nuget-hosted)
 * - When the name is different from the [production release] branch
 *   builds from this branch produce packages with the branchname
 *   added to the end of the packagename
 *
 * Accessed in pipeline as: "$projectReleaseBranch"
 */
def getProjectReleaseBranch() {
    return 'release'
}

/* Returns the name of the project-specific integration branch
 * The Project integration branch is a branch used to create releases for
 * deployment on [SBX/DEV] servers.
 * - [production release] is protected so an integration branch MUST exist.
 *   It is perfectly fine for this branch to be used by a lone developer
 *   providing application support instead of a team adding features
  * - MAY BE THE SAME NAME AS THE [production integration] BRANCH.
 *   Projects that do not conflict with ongoing PAS team support releases may
 *   use the [production integration] branch as the [project integration]
 *   branch. In those cases set the name of the [project integration] branch
 *   to be the same as the [production integration] branch. This will
 *   allow creation of snapshot installation packages without a branchname
 *   added to the end of the packagename. Those packages may be used to upgrade
 *   an existing installation instead of running the project side-by-side with
 *   an actively supported PAS version
 *
 * BEHAVIORS:
 * - Builds from this branch are pushed to Nexus (nuget-snapshots)
 * - Builds from this branch produce packages with the branchname
 *   added to the end of the packagename
 *
 * Accessed in pipeline as: "$projectIntegrationBranch"
 */
def getProjectIntegrationBranch() {
    return 'integration'
}

def getSandboxBranch1(){
	return 'sandbox_pudl'
}

def getSandboxBranch2(){
	return 'sandbox_advc'
}

def getProjectIntegrationAdvcBranch() {
    return 'integration_advc'
}
/////////////////////////////////////////////////////////////////////////////////


pipeline {
    // agent section specifies where the entire Pipeline will execute in the Jenkins environment
    agent {
        /**
         * node allows for additional options to be specified (you can also specify label '' without the node option)
         * if you want to execute the pipeline on any available agent use the option 'agent any'
         */
        node {
            label 'Windows' // Execute the Pipeline on an agent available in the Jenkins environment with the provided label(s)
        }
    }

    // configure Pipeline-specific options
    options {
        // Require an explicit checkout command in a stages
        skipDefaultCheckout()

        // keep only last 10 builds
        buildDiscarder (
            logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
        )
        // timeout job after 60 minutes
        timeout (
            time: 60,
            unit: 'MINUTES'
        )
        // Teams connection to "Software Development Project Team Online - Automated Builds" Channel
        office365ConnectorWebhooks([
            [
                name: '$PROJECT_NAME - v${major_version_num}.${release_num}.${BUILD_NUMBER} - $BUILD_STATUS!',
                notifyBackToNormal: true,
                notifyFailure: true,
                notifySuccess: true,
                notifyUnstable: true,
                url: 'https://outlook.office.com/webhook/95c9cb77-2215-414e-8b19-d92f6d77af05@b61c7997-54b1-4f99-82c7-d392d0d4a7f1/JenkinsCI/60ad43bc591644fe818466a295e75199/88078726-9594-43fe-b433-80f8ddba6f9c'
            ]
        ])

        // Set the URL for the GitHub project option
        githubProjectProperties(
            projectUrlStr: 'https://wpv-github.fhlbi.com/ApplicationsDevelopment-Active/CSCoreSvc-Credit-API'
        )
    }

    triggers {
      githubPush() // GitHub hook trigger for GITScm polling
    }

    /**
     * parameters directive provides a list of parameters which a user should provide when triggering the Pipeline
     * some of the valid parameter types are booleanParam, choice, file, text, password, run, or string
     */
    parameters {
        string (
            name: 'notify_on_complete',
            defaultValue: "$emailRecipients",
            description: 'comma-separated list of email recipients who should be notified when the build attempt completes',
            trim: false
        )
        string (
            name: 'major_version_num',
            defaultValue: params.major_version_num ?: '1',
            description: 'Major version number for the build',
            trim: false
        )
        string (
            name: 'release_num',
            defaultValue: params.release_num ?: '0',
            description: 'Release number, used for the minor number on the build version',
            trim: false
        )
    }

    environment {
        BUILD_TOOL = "%WORKSPACE%\\Microsoft\\dotnet\\dotnet.exe"
        DEF_BUILD_PARAMS = "--configuration Release --no-restore"
        DOTNET_SDK_VERSION = "5.0.202"
        PACKAGE_TITLE = "FHLBI Credit Suite API"
        PACKAGE_DESCRIPTION = "NuGet formatted package for Credit Services API - This is a .NET 5 (Core) Web API."
        BuildToolsLocation = "D:\\BuildTools"
    }

    /**
     * stages contain one or more stage directives
     */
    stages {
        /**
         * the stage directive should contain a steps section, an optional agent section, or other stage-specific directives
         * all of the real work done by a Pipeline will be wrapped in one or more stage directives
         */
        stage('Prepare') {
            steps {
                script {
                    // On first build by user just load the parameters as they
                    // are not available on first run of new branches
                    if (env.BUILD_NUMBER.equals("1") && currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause') != null)
                    {
                        currentBuild.displayName = 'Parameter loading'
                        addBuildDescription('Please restart pipeline')
                        currentBuild.result = 'ABORTED'
                        error('Stopping initial manually triggered build as we only want to get the parameters')
                    }
                }

                script {
                    def scmInfo = checkout scm

                    // Multibranch pipelines do not begin with 'origin/' but regular pipelines do
                    env.REPO_BRANCHNAME = scmInfo.GIT_BRANCH.minus('origin/')
                    
                    // Multibranch pipelines do not assign the GIT_URL but regular pipelines do
                    if (env.GIT_URL == null)
                    {
                        echo 'Assigning GIT_URL using scmInfo'

                        env.GIT_URL = scmInfo.GIT_URL
                    }
                }

                script {
                    // Setup the initial value for PACKAGE_VERSION outside of the
                    // environment block to allow reassignment if necessary. Values
                    // set in the environment block of a script can't be altered
                    env.PACKAGE_VERSION = "${params.major_version_num}.${params.release_num}.${env.BUILD_NUMBER}"

					// Builds from any branch other than the [production release] branch will
					// always append the branchname to the package name to prevent package
					// name & version conflicts
					env.PACKAGE_SUFFIX = ''

                    // Setup the tag that will be used in GitHub to mark the
                    // source of a release candidate build for QA/production
                    env.SOURCE_TAG = "$tagPrefix" + "." + env.PACKAGE_VERSION

					if (env.REPO_BRANCHNAME != "$productionReleaseBranch")
					{
						env.SOURCE_TAG = env.REPO_BRANCHNAME + "." + env.PACKAGE_VERSION
						
                        if (env.REPO_BRANCHNAME != "$productionIntegrationBranch")
                        {
						    env.PACKAGE_SUFFIX = "." + env.REPO_BRANCHNAME
					    }
                    }
                }

                powershell label: 'Prepare the dotnet SDK for use',
                    script: '''
                        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

                        $scriptpath = $ENV:BuildToolsLocation

                        # temporarily change to the script path folder
                        Push-Location $scriptpath

                        # Run the dotnet install script to get the SDK
                        ./dotnet-install.ps1 -Version $ENV:DOTNET_SDK_VERSION -NoCdn -UncachedFeed $ENV:NexusInstance/repository/MicrosoftBinaries/dotnet -InstallDir $ENV:WORKSPACE\\Microsoft\\dotnet

                        # return to the workspace
                        Pop-Location
                    '''
            }
        }
		stage('Tag source for Release') {
            // when directive allows the Pipeline to determine whether the stage should be executed depending on the given condition
            // built-in conditions - branch, expression, allOf, anyOf, not etc.
            when {
                expression {
                    return (env.REPO_BRANCHNAME == "$productionReleaseBranch" || env.REPO_BRANCHNAME == "$projectReleaseBranch")
                }
            }
            steps {
                echo "Triggering tag for branch (${env.SOURCE_TAG}, ${env.REPO_BRANCHNAME}, ${env.GIT_URL})"

                script {
                    try {
                        build job: 'Action - Tag an FSDApp for Release (parameterized)',
                            parameters: [
                                string(
                                    name: 'fhlbi_build_label',
                                    value: env.SOURCE_TAG
                                ),
                                string(
                                    name: 'fhlbi_branch',
                                    value: 'origin/' + env.REPO_BRANCHNAME
                                ),
                                string(
                                    name: 'fhlbi_repository',
                                    value: env.GIT_URL
                                )
                            ],
                            wait: true
                    } catch (err) {
                        echo err.getMessage()
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
		}
		stage('SAST Scan') {
            when {
                /* Execute the stage when the specified Groovy expression evaluates to true
                 */
                expression {
                    return (env.REPO_BRANCHNAME == "$productionReleaseBranch" || env.REPO_BRANCHNAME == "$projectReleaseBranch")
                }
            }
            steps {
                step([
                    $class: 'CxScanBuilder',
                    avoidDuplicateProjectScans: true,
                    comment: '',
                    exclusionsSetting: 'global',
                    incremental: false,
                    fullScanCycle: 10,
                    groupId: 'f5b1a849-ab6c-4042-9301-5301dcd82579',
                    preset: '100004',
                    projectName: "$srcProjectName",
                    sastEnabled: true,
                    sourceEncoding: '1',
                    waitForResultsEnabled: true
                ])
            }
	}	
		stage('Build') {
            steps {
                // If a PACKAGE_SUFFIX is specified then find and replace the name of the package in the nuspec files
                script {
                    if (env.PACKAGE_SUFFIX != '')
                    {
                        powershell label: 'Rename non-production NuGet package(s)',
                    script: """
                        (Get-Content .\\FHLBI.CreditSuite.Api\\FHLBI.CreditSuite.Api.csproj).replace(\'<PackageId>\$(AssemblyName)</PackageId>\', \'<PackageId>\$(AssemblyName)${env.PACKAGE_SUFFIX}</PackageId>\') | Set-Content .\\FHLBI.CreditSuite.Api\\FHLBI.CreditSuite.Api.csproj
                        """
                    }
                }

                script {                    
				    // Release versions use the default configuration for NuGet. All others use the snapshot configuration
                    def NuGetConfig = env.NugetInstallLocation + '\\NuGet-Snapshots.Config';
                    if (env.REPO_BRANCHNAME == "$productionReleaseBranch" || env.REPO_BRANCHNAME == "$projectReleaseBranch")
                    {
                        NuGetConfig = env.NugetInstallLocation + '\\NuGet.Config';
                    }

                    // Release versions need to ensure that snapshot packages are not used in builds.
                    // Snapshot will use the default packages directory (%userprofile%\\.nuget\\packages\\)
                    def NuGetLocalPackageLocation = '';
                    if (env.REPO_BRANCHNAME == "$productionReleaseBranch" || env.REPO_BRANCHNAME == "$projectReleaseBranch")
                    {
                        NuGetLocalPackageLocation = 'SET NUGET_PACKAGES=%userprofile%\\.nuget\\packages-hosted\\';
                    }
                    
					// dotnet.exe does not allow packages to be updated using a
                    // --configfile parameter. The workaround is to copy the
                    // appropriate NuGet.Config into the solution directory and
                    // ensure that only the specific sources in that file are
                    // used by "clearing" all previously detected ones using a
                    // <clear> command in the packageSources.
                    // NOTE: Future updates to Nuget.config in Ansible may make
                    // the XML editing part of this step unnecessary
                    powershell label: 'Prepare to update referenced packages',
                        script: """
                            \$solution_nuget = (Join-Path ${env.WORKSPACE} 'NuGet.Config')

                            # Copy the NuGet.Config in use for this build
                            Copy-Item ${NuGetConfig} -Destination \$solution_nuget

                            # Add the <clear> node
                            \$xmlfile = [XML](Get-Content \$solution_nuget)

                            \$clear_element = \$xmlfile.CreateElement("clear")
                            \$xmlfile.configuration.packageSources.PrependChild(\$clear_element)

                            \$xmlfile.save(\$solution_nuget)
                        """


                    // Restore packages separately in case an update is needed before the actual build is run.
                    // 'dotnet build' also attempts to retore packages.
                    bat label: 'Restore packages for solution',
                        script: """
                            ${NuGetLocalPackageLocation}
                            \"${BUILD_TOOL}\" restore \"%WORKSPACE%\\FHLBI.CreditSuite.sln\" --configfile ${NuGetConfig}
                        
							ECHO Ensuring Packages are set to latest version...
                            pushd FHLBI.CreditSuite.Api
                            \"${BUILD_TOOL}\" add package FHLBI.Infrastructure.ApiRequestResponseUtility
                            popd

                            pushd FHLBI.CreditSuite.Common
                            \"${BUILD_TOOL}\" add package FHLBI.Infrastructure.ApiRequestResponseUtility
                            \"${BUILD_TOOL}\" add package FHLBI.Infrastructure.ExceptionService
                            \"${BUILD_TOOL}\" add package FHLBI.Infrastructure.LogService
                            popd
						
						"""

                    slnName = "FHLBI.CreditSuite.Api\\FHLBI.CreditSuite.Api.csproj"
                    
                    bat label: 'Build & Publish Credit Suite API',
                        script: "\"${BUILD_TOOL}\" publish \"${slnName}\" ${DEF_BUILD_PARAMS} -p:Version=${env.PACKAGE_VERSION} --output \"${env.WORKSPACE}\\LatestBuild\\\\\" --configfile ${NuGetConfig}"

                    bat label: 'Pack FHLBI.CreditSuite.Api',
                        script: """
                            ${env.BuildToolsLocation}\\octo.exe pack --id=FHLBI.CreditSuite.Api${env.PACKAGE_SUFFIX} --version=${env.PACKAGE_VERSION} --title=\"${env.PACKAGE_TITLE}\" --description=\"${env.PACKAGE_DESCRIPTION}\" --basePath=\"${env.WORKSPACE}\\LatestBuild\\\\\" --outFolder .
                        """
                }
            }
        }
        stage('Test') {
            steps {
                echo "No Automated Tests"
            }
        post {
                // Only run the steps if the current Pipeline's or stage's run has a "success" status
                success {
                    script {
                        // Releases intended for deployment will be pushed to Nexus
                        if (env.REPO_BRANCHNAME == "$productionReleaseBranch"
                            || env.REPO_BRANCHNAME == "$projectReleaseBranch"
                            || env.REPO_BRANCHNAME == "$projectIntegrationBranch"
                            || env.REPO_BRANCHNAME == "$productionIntegrationBranch"
			    || env.REPO_BRANCHNAME == "$sandboxBranch1"
			    || env.REPO_BRANCHNAME == "$sandboxBranch2"
			     || env.REPO_BRANCHNAME == "$projectIntegrationAdvcBranch")
                        {
                            // Release versions use the hosted repository. All others use the snapshot repository
                            def RepositoryName = 'nuget-snapshots';
                            if (env.REPO_BRANCHNAME == "$productionReleaseBranch" || env.REPO_BRANCHNAME == "$projectReleaseBranch")
                            {
                                RepositoryName = 'nuget-hosted';
                            }
                    
                            withCredentials([string(credentialsId: 'jenkins-build-user', variable: 'API_KEY')]) {
                                bat label: "Push NuGet packages to ${RepositoryName} repository",
                                    script: """
                                        @ECHO OFF
                                         \"${BUILD_TOOL}\" nuget push "%WORKSPACE%/FHLBI.CreditSuite.Api${env.PACKAGE_SUFFIX}.${env.PACKAGE_VERSION}.nupkg" -s %NexusInstance%/repository/${RepositoryName}/ -k %API_KEY%
                                    """
                            }
                        }
                    }
                }
            }
        }
    } 

    /**
     * post section defines actions which will be run at the end of the Pipeline run or stage
     * post section condition blocks: always, changed, failure, success, unstable, and aborted
     */
    post {
        // Run regardless of the completion status of the Pipeline run
        always {
            emailext body: '$DEFAULT_CONTENT',
                    mimeType: 'text/html',
                    postsendScript: '$DEFAULT_POSTSEND_SCRIPT',
                    presendScript: '$DEFAULT_PRESEND_SCRIPT',
                    replyTo: '$DEFAULT_REPLYTO',
                    subject: '$PROJECT_NAME - v' + env.PACKAGE_VERSION + ' - $BUILD_STATUS!',
                    to: "${params.notify_on_complete}"   
        }
	}
}
