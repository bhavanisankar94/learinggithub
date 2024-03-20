name: CI/CD Pipeline

# Controls when the workflow will run
on:
  push:
    branches: [ "CI_CD_Github_Actions" ]
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    # Checks-out your repository
    - uses: actions/checkout@v3
      with:
        ref: 'CI_CD_Github_Actions'
    
    # Gradle build
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'
        
    - name: Gradle Build
      run: |
        java -version
        ./gradlew -version
        cd mongo_rest && ./gradlew :safetraxrest:distzip
        ls mongo_rest/safetraxrest/build/distributions

    # Store artifacts in GCP
    - name: Store Artifacts in GCP
      run: |
        gsutil cp mongo_rest/safetraxrest/build/distributions/safetraxrest.zip gs://test-live-deploy/SERVERV2
      env:
        GOOGLE_APPLICATION_CREDENTIALS: ${{ secrets.GCP_SA_KEY }}

    # Deploy to Another Server in GCP
    - name: Deploy to Another Server
      run: |
        ssh -i ${{ secrets.SSH_PRIVATE_KEY }} -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no venu@cicd-test "sudo /home/venu/ETS-mongoser-deployment-updated.sh"
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
