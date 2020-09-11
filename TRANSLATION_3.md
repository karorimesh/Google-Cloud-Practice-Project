# Creating Virtual Machines
## Create a utility virtual machine
	gcloud beta compute --project=qwiklabs-gcp-00-70ff213abbfa instances create instance-1 --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --no-address --maintenance-policy=MIGRATE --service-account=1028945641688-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-1 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

## Create a Windows virtual machine
	gcloud beta compute --project=qwiklabs-gcp-00-70ff213abbfa instances create instance-2 --zone=europe-west2-a --machine-type=n1-standard-2 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1028945641688-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --image=windows-server-2016-dc-core-v20200908 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd --boot-disk-device-name=instance-2 --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any

	gcloud compute --project=qwiklabs-gcp-00-70ff213abbfa firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

	gcloud compute --project=qwiklabs-gcp-00-70ff213abbfa firewall-rules create default-allow-https --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:443 --source-ranges=0.0.0.0/0 --target-tags=https-server

### Set the password for the VM
	gcloud compute reset-windows-password instance-2
	**_Press 'Y' when prompted_**
	**_Fill in the required fields to set the password_**

## Create a custom virtual machine
	gcloud beta compute --project=qwiklabs-gcp-00-70ff213abbfa instances create instance-3 --zone=us-west1-b --machine-type=e2-custom-6-32768 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=1028945641688-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=debian-10-buster-v20200910 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=instance-3 --no-shielded-secure-boot --no-shielded-vtpm --no-shielded-integrity-monitoring --reservation-affinity=any

### Connect via SSH to your custom VM
	gcloud compute ssh instance-3 --zone us-central1-c --tunnel-through-iap
	free
	sudo dmidecode -t 17
	nproc
	lscpu
	exit