

# Prep Edge OCP (on Laptop or Edge Server)
- Install OCP Local or SNO on Edge gateway. Note: Make sure at least 16GB RAM
- Copy kubeconfig from Robot Microshift: `/var/lib/microshift/resources/kubeadmin/kubeconfig` to admin laptop
- In kubeconfig replace 127.0.0.1 with Robot IP
- Install OCP GitOps on edge OCP
- Install Argo Cli on admin laptop
- Get ArgoCD admin credentials (non SSO) from secret `openshift-gitops-cluster`

# Configure GitOps
- Login to ArgoCD Web UI
- Login argocd client on admin laptop with creds from secret:
  - _argocd login openshift-gitops-server-openshift-gitops.apps-crc.testing:443_
- Add Microshift to GitOps:
  - `argocd cluster add microshift --kubeconfig=<kubeconfig-file`
- Verify Microshift shows up in ArgoCD UI: Settings->Clusters
- Create Application in ArgoCD UI:
  - Applications->Create Application
  - Application Name: starterapp
  - Project Name: Default
  - Sync Policy: Automatic
  - Repository URL: https://github.com/cloud-native-robotz-hackathon/devel-bucket.git
  - Path: gitops-starterapp-python
  - Cluster URL: <Choose Microshift>
  - Namespace: Terminator (needs to be created before)

# Issues
- Name resolution for accessing robot webpage