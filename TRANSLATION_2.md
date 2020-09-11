# Running AI models on Kubeflow
## Set up the environment
	gcloud auth list
	gcloud config list project
	**_Enable Boost mode_**

## Download the project files to Cloud Shell
	git clone https://github.com/GoogleCloudPlatform/training-data-analyst
	cd training-data-analyst/courses/data-engineering/kubeflow-examples

## Setting Environment Variables
	export PROJECT_ID=$(gcloud config list project --format "value(core.project)")
	gcloud config set project $PROJECT_ID
	export ZONE=us-central1-a
	gcloud config set compute/zone $ZONE
	cd ./mnist
	WORKING_DIR=$PWD

## Installing Kustomize
	mkdir $WORKING_DIR/bin
	wget https://storage.googleapis.com/cloud-training/dataengineering/lab_assets/kubeflow-resources/kustomize_2.0.3_linux_amd64 -O $WORKING_DIR/bin/kustomize
	chmod +x $WORKING_DIR/bin/kustomize
	PATH=$PATH:${WORKING_DIR}/bin

### Install kfctl
	wget -P /tmp https://storage.googleapis.com/cloud-training/dataengineering/lab_assets/kubeflow-resources/kfctl_v1.0-0-g94c35cf_linux.tar.gz
	tar -xvf /tmp/kfctl_v1.0-0-g94c35cf_linux.tar.gz -C ${WORKING_DIR}/bin

## Enabling the API
	gcloud services enable container.googleapis.com

## Create a cluster
	export KUBEFLOW_USERNAME=<REPLACE_WITH_QWIKLABS_USERNAME>
	export KUBEFLOW_PASSWORD=<REPLACE_WITH_QWIKLABS_PASSWORD>
	export CONFIG_URI=https://storage.googleapis.com/cloud-training/dataengineering/lab_assets/kubeflow-resources/kfctl_gcp_basic_auth.v1.0.1.yaml
	export KF_NAME=kubeflow
	export KF_DIR=${WORKING_DIR}/${KF_NAME}
	mkdir -p ${KF_DIR}
	cd ${KF_DIR}
	kfctl build -V -f ${CONFIG_URI}
	sed -i 's/n1-standard-8/n1-standard-4/g' gcp_config/cluster-kubeflow.yaml
	sed -i 's/1.14/1.15.12-gke.2/g' gcp_config/cluster-kubeflow.yaml
	export CONFIG_FILE=${KF_DIR}/kfctl_gcp_basic_auth.v1.0.1.yaml
	kfctl apply -V -f ${CONFIG_FILE}
	gcloud container clusters get-credentials ${KF_NAME} --zone ${ZONE} --project ${PROJECT_ID}
	kubectl config set-context $(kubectl config current-context) --namespace=kubeflow
	kubectl get all

## Training
### Setting up a Storage Bucket
	BUCKET_NAME=${KF_NAME}-${PROJECT_ID}
	gsutil mb gs://${BUCKET_NAME}/

### Building the Container
	IMAGE_PATH=us.gcr.io/$PROJECT_ID/kubeflow-train
	docker build $WORKING_DIR -t $IMAGE_PATH -f $WORKING_DIR/Dockerfile.model
	docker run -it $IMAGE_PATH

	**_Press CTRL+C_**
	gcloud auth configure-docker --quiet
	docker push $IMAGE_PATH

## Training on the Cluster
	cd $WORKING_DIR/training/GCS
	kustomize edit add configmap mnist-map-training \
    --from-literal=name=my-train-1
    kustomize edit add configmap mnist-map-training \
    --from-literal=trainSteps=200
	kustomize edit add configmap mnist-map-training \
    --from-literal=batchSize=100
	kustomize edit add configmap mnist-map-training \
    --from-literal=learningRate=0.01
    kustomize edit set image training-image=${IMAGE_PATH}:latest
	kustomize edit add configmap mnist-map-training \
    --from-literal=modelDir=gs://${BUCKET_NAME}/my-model
	kustomize edit add configmap mnist-map-training \
    --from-literal=exportDir=gs://${BUCKET_NAME}/my-model/export
    gcloud --project=$PROJECT_ID iam service-accounts list
    kubectl describe secret user-gcp-sa
    kustomize edit add configmap mnist-map-training \
    --from-literal=secretName=user-gcp-sa
	kustomize edit add configmap mnist-map-training \
    --from-literal=secretMountPath=/var/secrets
	kustomize edit add configmap mnist-map-training \
    --from-literal=GOOGLE_APPLICATION_CREDENTIALS=/var/secrets/user-gcp-sa.json
    sed -i 's/default-editor/kf-user/g' ../**/*
    kustomize build .
    kustomize build . | kubectl apply -f -
    kubectl describe tfjob

    **_Press CTRL+C_**

    kubectl logs -f my-train-1-chief-0
    gsutil ls -r gs://${BUCKET_NAME}/my-model/export
    cd $WORKING_DIR/serving/GCS
    kustomize edit add configmap mnist-map-serving \
    --from-literal=name=mnist-service
    kustomize edit add configmap mnist-map-serving \
    --from-literal=modelBasePath=gs://${BUCKET_NAME}/my-model/export
    sed -i 's/default-editor/kf-user/g' ../**/*
    kubectl describe service mnist-service

## Deploying the UI
	cd $WORKING_DIR/front
	kustomize build . | kubectl apply -f -
	kubectl port-forward svc/web-ui 8080:80
	**_View the page using web preview on port 8080_**