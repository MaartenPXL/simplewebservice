podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: maven
        image: maven:3-openjdk-11
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
          - name: m2
            mountPath: /root/.m2
      - name: kaniko
        image: gcr.io/kaniko-project/executor:debug
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
          - name: kaniko-secret
            mountPath: /kaniko/.docker
          - name: dockersock
            mountPath: /var/run/docker.sock
      restartPolicy: Never
      volumes:
      - name: kaniko-secret
        secret:
            secretName: dockercred
            items:
            - key: .dockerconfigjson
              path: config.json
      - name: dockersock
        hostPath:
          path: /var/run/docker.sock
      - name: m2
        hostPath:
          path: /root/.m2
''') {
  node(POD_LABEL) {
    stage('Maven: Retrieve project') {
      git url: 'https://github.com/systemsexpert-pxl/simplewebservice.git', branch: 'main'
      container('maven') {
        stage('Maven: Build project') {
          sh '''
            mvn clean package
          '''
        }
      }
    }

    stage('kaniko: Build & Deploy Image') {
      container('kaniko') {
        stage('Build & Deploy to dockerhub') {
          sh '''
            /kaniko/executor --context `pwd` --destination tomcoolpxl/testje:1.0
          '''
        }
      }
    }

  }
}