#!groovy

def uploadAndInvalidate() {
  def s3Path = "/codelabs"
  uploadAssetsToS3('dist', "s3://codelabs-prod/tutorials/", 'us-east-1', true, false, 86400, 'prod');
  invalidateCDN('E2OJAP80EGS5J2', '\"/*\"', 'prod');
}

pipeline {
    agent any
    stages {
        stage('upload to s3') {
            steps {
                uploadAndInvalidate();
            }
        }
    }
}
