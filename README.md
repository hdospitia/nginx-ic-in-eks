# NGINX in EKS

## Key aspects

- Take care of the required NGINX version: OSS or Plus.
- Validate if your architecture requires CRDs.
- If you still want to use Classic LB (After the recommendation from NGINX team), enable PROXYv2 protocol to get client connection details. It should be enabled in both the Ingress Controller (IC) and the Classic Load Balancer. Note that this not apply to NLB, which does not need or provides the PROXY protocol.
- For AWS, the recommendation is to use NLB instead of the default Classic LB. Configure an annotation in the IC Service to enable the NLB.
- NLB allows to manage non HTTP/S traffic with NGINX.
- To make adjustments like above, it is preffered to pull the Helm Chart and set the right values in the values.yaml file.

## Installation

Helm from registry (and for this excercise):

    CHART_VERSION=1.2.1 helm install -n ngxinx-ingress-controller-system --create-namespace nginx-ic oci://ghcr.io/nginxinc/charts/nginx-ingress --version $CHART_VERSION --values values.yaml

Helm from local. Dont forget to update the values with below configuration if you are still using the default Classic Load Balancer:

    controller.service.annotations: service.beta.kubernetes.io/aws-load-balancer-type: nlb
    controller.config.entries:
        proxy-protocol: "True"
        real-ip-header: "proxy_protocol"
        set-real-ip-from: "0.0.0.0/0"

Pull and install Helm Chart:
    
    helm pull oci://ghcr.io/nginxinc/charts/nginx-ingress --untar --version 1.2.1
    cd nginx-ingress
    helm install nginx-ic . -n nginx-ic-system --create-namespace


# Test with sample app

Namespace:

    kubectl create ns app

Workload:

    kubectl -n app apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/main/examples/ingress-resources/complete-example/cafe.yaml

Secret:

    kubectl -n app apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/main/examples/ingress-resources/complete-example/cafe-secret.yaml

Ingress:

    kubectl -n app apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/main/examples/ingress-resources/complete-example/cafe-ingress.yaml

Clean up:

    kubectl delete ns app --grace-period=0 --force
    helm uninstall -n nginx-ic-system nginx-ic
    kubectl delete ns nginx-ic-system

# Documentation

- [NGINX Ingress Controller](https://docs.nginx.com/nginx-ingress-controller/)
- [NGINX Ingress Controller in Kubernetes](https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/)
- [EKS and NLB with NGINX](https://aws.amazon.com/blogs/opensource/network-load-balancer-nginx-ingress-controller-eks/)