# redredred
redredred

yum install httpd-tools
yum install htpasswd

htpasswd -c -B -b ex280-users leader redhat
htpasswd -B -b ex280-users developer redhat
htpasswd -B -b ex280-users tester redhat
htpasswd -B -b ex280-users jobs redhat
htpasswd -B -b ex280-users qa redhat

oc create secret generic ex280-secret --from-file htpasswd=ex280-users -n openshift-config

oc get oauth cluster -o yaml > oauth.yaml

vim oauth.yaml
---
spec: 
  identityProviders:
  - name: ex280-users
    mappingMethod: claim
	type: HTPasswd
	  fileData:
	     name: ex280-secret

oc replace -f oauth.yaml
========================================================
oc adm policy add-cluster-role-to-user cluster-admin jobs
oc get clusterrolebindings | grep self
oc adm policy add-cluster-role-to-user self-provisioner developer

oc adm policy  remove-cluster-role-to-user self-provisioner system:authenticated:oauth

oc login -u jobs -p redhat
oc get node

oc get secret -n kube-system| grep kubeadmin
#oc delete secret kubeadmin -n kube-system
========================================================
oc new-project project1
oc new-project project2
oc new-project project3
oc new-project project4
oc new-project project5

oc adm policy add-role-to-user admin leader -n project1
oc adm policy add-role-to-user admin leader -n project2

oc adm policy add-role-to-user admin developer -n project2
oc adm policy add-role-to-user view qa -n project3

========================================================
oc adm groups new production
oc adm groups new development
oc adm groups new testing

oc adm groups add-users production leader
oc adm groups add-users development developer qa
oc adm groups add-users testing tester

oc policy add-role-to-group edit development -n project4
oc policy add-role-to-group view testing -n project5

========================================================
oc project quota-<name>
oc create quota ex280-quota --hard memory=1Gi,cpu=1,pods=5,services=5,replicationControllers=5
========================================================
oc project limits-<name>
oc describe limits

vim ex280-limits.
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "ex280-quotalimits"
spec: 
  limits:
    - type: "Pod"
	  max:
	    cpu: "500m"
		memory: "500Mi"
	  min:
	    cpu: "5m"
		memory: "300Mi"
	- type: "Container"
	  max: 
	    cpu: "500m"
		memory: "500Mi"
	  min: 
	    cpu: "5m"
		memory: "300Mi"
	  defaultRequest:
	     cpu: "300m"
		 memory: "400Mi"

oc create -f ex280-limits.yaml
oc describe limits
=======================================================
oc project schedule
oc get pods
oc get events
oc get nodes
oc edit node workernode01
remove 4 lines under specifications.
spec:
  taints:
  - effect: NoExecute
    key: key1
    value: value1
(Delete Taints and 3 lines)

oc get pod ( running )

oc get route -n schedule
curl <route_path>
Hello Openshift !!!!

oc delete route/<hello> -n schedule
oc get service 
oc expose service/<hello> --hostname=<given-url-in question>
oc get route
curl hello.app.gls.test

========================================================
oc project dev-<name>
oc get pods
oc scale dc/scale-<uname> --replicas=5
oc get dc

=======================================================
oc project prod-<uname>
oc get pods
oc get dc

oc edit dc ----- then also spec - image - resources: requests: cpu: 25m limits: cpu: 100m

oc export -o yaml > auto-<uname>.yaml
vim auto-<uname>.yaml
spec:
  containers:
  - name: xxxxxxx openshift/hellow-openshift
    name: project5-<uname>
	resources:
	  requests: 
	    cpu: "25m"
	  limits:
	    cpu: "100m"

oc replace -f auto-<uname>.yaml
oc autoscale dc/autoscale-<uname> --min=6 --max 10 --cpu-percent 80

oc get hpa
oc get pods
=======================================================
openssl genrsa -out private.key 2048
openssl req -new -key private.key -out web.csr -subj "/C=US/ST=CA/L=Los Angeles/O=Example/OU=IT/CN=secure-
<urname>.apps.gls.test"

openssl x509 -req -in web.csr -signkey private.key -out public.crt

oc get service
oc get route
oc delete route secure-<uname> 

oc create route edge --service=secure-<uname> --cert=public.crt --key=private.key --hostname=secure-<uname>.apps.gls.test

curl -k https://secure.<uname>.apps.gls.test
========================================================
oc project simla
oc create sa ex280-sa
oc get scc
oc adm policy add-scc-to-user anyuid -z ex280-sa

========================================================
oc project simla		
oc get pods
oc get events
oc get events --field-selector type=Warning
oc get dc
oc get pod
oc logs oranges-1-72wh6
oc get dc
oc set sa dc/oranges ex280-sa
oc get dc
oc get pods
oc get routes

oc describe service

oc logs <pod name>
Cannot create directory[/etc/gitlab] at /etc/gitlab due to insufficient permissions
oc set serviceaccount dc gitlab-<urname> sa-<urname>
oc get pods
Note:
Now pods in running state , but unable to browse
oc get events --field-selector type=Warning

oc get pod --show-labels
oc edit svc/oranges
spec:
selector:
app: orange
Note: add s to orange based on error msg, Now you can acces

oc get route
oc delete route/oranges
oc get svc
oc get route
oc describe svc
oc get pod --show-labels
oc get route
curl oranges.apps.gls.com
oc delete route/oranges
oc get srv
oc expose svc/oranges --hostname=oranges.apps.gls.com
oc get route

========================================================

oc project secret-<uname>
oc create secret generic ex280-secret --from-literal
mysql_root_password=r3dh4t123

========================================================
14) Apply secret
with Name MYSQL_ROOT_PASSWORD in secret-<urname>

oc get pods
oc status --suggest
oc logs <pod-name>
oc get secret
oc get dc
oc set env dc/sql --from secret/ex280-secret
oc get pods

========================================================
15) Deploy

oc project tokyo-project
oc get pods
oc get events
oc get events --field-selector type=Warning
oc describe pod/math-<uname>
Identity nodeselector name (case sensitive) (don't modify)

oc get nodes
oc get node master-1.gls.test --show-labels
Identity nodeselector name (case sensitive)
Now set label name for worker nodes
oc label node worker-1.gls.test star=Trek --overwrite
oc label node master-1.gls.test star=Trek --overwrite
Now check route
oc get route
In Exam, check url to access if route and given url is same ..no need to modify
otherwise
oc edit route <route-name>
2 locations we need to modify FQDN as per given url --> spec:host and
ingress:host

curl math.aps.gls.test
=======================================================
16) Deploy

oc project china
oc get events
oc get events --field-selector type=Warning
oc get pods
oc logs <pod-name> --> no luck
oc describe pod <pod-name>
We can find issue with warnings
oc edit dc <dc-name>   
find & change memory 80GiB or 60 GiB to 80Mi / 60 Mi
oc get pod check pod is running
oc get route
check access
======================================================

