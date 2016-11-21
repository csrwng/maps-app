# Maps Application Example

Integrated pipeline for micro-services application

To instantiate:

```
oc cluster up
oc new-project integration
oc login -u system:admin
oc adm policy add-cluster-role-to-user self-provisioner system:serviceaccount:integration:jenkins
oc login -u developer
oc new-app https://github.com/csrwng/maps-app.git
```
