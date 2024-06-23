
pipeline {
    agent any
    environment {
        BRANCH_NAME = "development"
        ENV = "LOWER"
        COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        TIME_STAMP = sh(returnStdout: true, script: 'date +%Y.%m.%d-%H.%M.%S').trim()
        DOCKER_TAG = ""
        DEFAULT_VERSION = 'Release-V1.0.0'
        NEW_TAG = ""
        OLD_TAG = ""
    }

    stages {

        stage('Pull') {
            steps {
               git branch: "${BRANCH_NAME}", url: 'https://github.com/codescapes-dev/CI.git'
            }
        }

        stage('Fetch Latest Release Tag') {
            steps {
                script {
                    try {
                        def latestTagOutput = sh(script: 'gh release view --json tagName', returnStdout: true).trim()
                        def json = readJSON text: latestTagOutput
                        OLD_TAG = json.tagName ?: DEFAULT_VERSION
                        NEW_TAG = bumpVersion(OLD_TAG)
                    } catch (Exception e) {
                        echo "No Prior Releases Found, Creating Default Release"
                        NEW_TAG = DEFAULT_VERSION
                    } finally {
                        echo "Latest Release Tag Is: ${NEW_TAG}"
                    }
                }
            }
        }

        stage ('Package Update') {
            steps {
                script {
                    sh "sed -i 's/\"version\": \".*\"/\"version\": \"${NEW_TAG}\"/' package.json"
                    sh "git add package.json"
                    sh "git commit -m 'Version Bump to ${NEW_TAG}'"
                    sh "git push origin development -u"
                }
            }
        }

        stage('Create PR') {
            steps {
                script {
                    sh "git prune"
                    sh "git fetch"
                    // echo "Searching and Closing All Pull Manual PRs opened on Target Branch"
                    // sh "gh pr list -s all --base main | awk "{print $1}" d | xargs -I{} gh pr close {}"
                    sh "gh pr create --fill --base main"
                    sleep time: 5, unit: 'SECONDS'
                    sh "gh pr merge --merge --auto"
                }
            }
        }

        stage('Create GH Release') {
            steps {
                script {
                    try {
                        echo "New: ${NEW_TAG}"
                        if (NEW_TAG == 'Release-V1.0.0') {
                            echo "If Case"
                            sh "gh release create ${NEW_TAG} --latest=true --generate-notes --target main"
                        } else {
                            echo "Else Case"
                            sh "gh release create ${NEW_TAG} --target main --notes-start-tag ${OLD_TAG} --generate-notes"
                        }
                    } catch (Exception e) {
                        echo "Error creating latest release tag: ${e.message}"
                    }
                }
            }
        }
    }
}

def bumpVersion(tag) {
    // Example: If tag is "Release-V1.1.25", extract the numeric parts and increment the version
    def matcher = tag =~ /(\d+)\.(\d+)\.(\d+)$/
    if (!matcher) {
        return tag // Return the original tag if no numeric parts are found
    }
    
    def major = matcher[0][1].toInteger()
    def minor = matcher[0][2].toInteger()
    def patch = matcher[0][3].toInteger()
    
    // Increment patch number
    patch += 1
    if (patch > 25) {
        patch = 0
        minor += 1
    }
    
    // If minor number exceeds 25, reset it and increment major number
    if (minor > 25) {
        minor = 0
        major += 1
    }
    
    def newVersion = "${major}.${minor}.${patch}"
    def newTag = tag.replaceFirst(/(\d+)\.(\d+)\.(\d+)$/, newVersion)
    return newTag
}
