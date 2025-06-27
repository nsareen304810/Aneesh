
pipeline {
  agent any

  environment {
    OPENSHIFT_PROJECT = 'airflow'
    AIRFLOW_IMAGE = 'quay.io/astronomer/ap-airflow:2.9.0-buster'
    KUBECONFIG_CRED_ID = 'openshift-kubeconfig'
  }

  stages {
    stage('Prepare') {
      steps {
        withCredentials([file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')]) {
          sh '''
            oc project $OPENSHIFT_PROJECT || oc new-project $OPENSHIFT_PROJECT
          '''
        }
      }
    }

    stage('Apply PVCs') {
      steps {
        sh 'oc apply -f manifests/airflow-pvcs.yaml'
      }
    }

    stage('Deploy PostgreSQL') {
      steps {
        sh 'oc apply -f manifests/postgres-deployment.yaml'
      }
    }

    stage('Airflow Config') {
      steps {
        sh 'oc apply -f manifests/airflow-config.yaml'
      }
    }

    stage('Init Airflow DB') {
      steps {
        sh '''
          oc delete job airflow-init-db --ignore-not-found
          oc apply -f manifests/init-airflow-db.yaml
          oc wait --for=condition=complete job/airflow-init-db --timeout=120s
        '''
      }
    }

    stage('Deploy Airflow Webserver') {
      steps {
        sh 'oc apply -f manifests/airflow-web.yaml'
      }
    }

    stage('Deploy Airflow Scheduler') {
      steps {
        sh 'oc apply -f manifests/airflow-scheduler.yaml'
      }
    }

    stage('Expose UI') {
      steps {
        sh 'oc get route airflow-web'
      }
    }
  }

  post {
    success {
      echo "Airflow deployed successfully in OpenShift!"
    }
    failure {
      echo "Something went wrong. Check pipeline logs."
    }
  }
}
