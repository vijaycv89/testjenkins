pipeline {
    agent any

    parameters {
        string(name: 'nexusUrl', description: 'Nexus Package URL')
        choice(name: 'templateType', choices: ['Mainframe_template_PRE', 'Mainframe_Restore_Template_PRE'], description: 'Select Template Type')
        string(name: 'releaseTaskNumber', description: 'Release Task Number')
        booleanParam(name: 'skipPreXrefValidation', defaultValue: true, description: 'Skip Pre-Xref Validation')
        booleanParam(name: 'skipReleaseXrefValidation', defaultValue: true, description: 'Skip Release Xref Validation')
    }

    stages {
        stage('Extract Information') {
            steps {
                script {
                    def regex = /.*\/(sgt|suk)-(\d+)\/(\d+)\/(\d+-\d+\.\d+\.\d+-rc\+fix)\/.*\.zip/
                    def matcher = nexusUrl =~ regex
                    if (matcher) {
                        def hostApplication = matcher[0][1]
                        def subApplication = matcher[0][2]
                        def version = matcher[0][4]

                        echo "Host Application: ${hostApplication}"
                        echo "Sub Application: ${subApplication}"
                        echo "Version: ${version}"
                        
                        def folderPath = "Corporate/Program_SW/PRE_Deployments/${hostApplication}/${subApplication}"
                        if (!fileExists(folderPath)) {
                            mkdir(folderPath)
                            echo "Folder created at: ${folderPath}"
                        } else {
                            echo "Folder already exists at: ${folderPath}"
                        }

                        // Store sub-application and version for renaming
                        env.SUB_APPLICATION = subApplication
                        env.VERSION = version
                    } else {
                        error "Unable to extract information from Nexus URL or 'rc' not found in URL"
                    }
                }
            }
        }

        stage('Copy Pipeline and Rename') {
            steps {
                script {
                    def templateName = params.templateType
                    def currentDate = new Date().format('yyyyMMdd_HHmm')
                    def releaseTaskNumber = params.releaseTaskNumber
                    
                    // Define the new pipeline name based on the naming convention
                    def newPipelineName = "${currentDate}_${env.SUB_APPLICATION}-${env.VERSION}_${templateName}_Release${releaseTaskNumber}"
                    
                    // Copy the pipeline without builds and rename it
                    renamePipeline(newPipelineName)
                }
            }
        }

        stage('Trigger Build') {
            steps {
                script {
                    // Prompt user to trigger build with parameters
                    input(message: 'Configure build parameters and trigger', parameters: [
                        booleanParam(name: 'deploy', defaultValue: true, description: 'Deploy'),
                        booleanParam(name: 'skipPreXrefValidation', defaultValue: params.skipPreXrefValidation, description: 'Skip Pre-Xref Validation'),
                        booleanParam(name: 'skipReleaseXrefValidation', defaultValue: params.skipReleaseXrefValidation, description: 'Skip Release Xref Validation')
                    ])
                }
            }
        }
    }
}

def fileExists(path) {
    def file = new File(path)
    return file.exists()
}

def renamePipeline(newName) {
    // Get the current job name
    def currentJobName = env.JOB_NAME
    
    // Copy the current job without builds
    cloneJobWithoutBuilds(currentJobName, newName)
    
    // Delete the old job
    deleteJob(currentJobName)
}

def cloneJobWithoutBuilds(oldJobName, newJobName) {
    // Run the system command to clone the job without builds
    sh "cp -r ${oldJobName} ${newJobName}"
}

def deleteJob(jobName) {
    // Run the system command to delete the old job
    sh "rm -rf ${jobName}"
}