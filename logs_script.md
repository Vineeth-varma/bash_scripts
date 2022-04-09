#! /bin/bash

touch kiali_logs.txt


kiali_pod=$(curl -k -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" -H "Content-Type: application/json" -H "Accept: application/json" https://${API_Server_IP}:${PORT}/api/v1/namespaces/istio-system/pods | jq -rM '.items[].metadata.name' | grep kiali |  awk 'NR == 1 { print $1 }')

curl -k -s --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" -H "Content-Type: application/json" -H "Accept: application/json" https://${API_Server_IP}:${PORT}/api/v1/namespaces/istio-system/pods/${kiali_pod}/log | sed '/TLS handshake error/d' > kiali_logs.txt

x=$(stat -c %s kiali_logs.txt)
y=1073741824


if [ "$x" -gt "$y" ];
then
        zip -r  "kiali_$(date +"%Y-%m-%d").zip" kiali_logs.txt
		rm kiali_logs.txt
else
        z=$(echo "scale=3; $x/$y" | bc)
        echo "file size is $z GB"
fi
