def buildNumber = env.BUILD_NUMBER

podTemplate(namespace: 'jenkins-ci', yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: kaniko-secret
          mountPath: /kaniko/.docker
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
''') {
  node(POD_LABEL) {
    stage('Build') {
      git url: 'https://github.com/nltimv/jenkins-pipeline-test.git', branch: 'main'
      container('kaniko') {
        stage('Build image') {
          sh '''
            /kaniko/executor --context "`pwd`" --destination nltimv/jenkins-pipeline-test:$BUILD_NUMBER --destination nltimv/jenkins-pipeline-test:latest
          '''
        }
      }
    }
  }
}
podTemplate(namespace: 'jenkins-ci', yaml: """
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: test
        image: nltimv/jenkins-pipeline-test:${buildNumber}
        command:
        - sleep
        args:
        - 9999999
        resources:
          requests:
            cpu: 200m
            memory: 512Mi
      restartPolicy: Never
""") {
  node(POD_LABEL) {
    stage('Test') {
      container('test') {
        stage('Call google.com using curl')
        sh '''
          curl https://google.com
        '''
      }
    }
  }
}