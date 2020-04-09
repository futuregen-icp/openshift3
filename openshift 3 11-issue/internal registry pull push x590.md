# internal registry pull,push x590

Created: Apr 09, 2020 10:05 PM
Tags: internal registry pullpush x590

tls-ca-bundle.pem
    cd /etc/pki/ca-trust/extracted/pem/
    
    oc create secret generic new-tls-ca-bundle --from-file=tls-ca-bundle.pem -n default
    
    $ oc secrets link registry new-tls-ca-bundle
    $ oc secrets link default new-tls-ca-bundle
    
    oc set volume dc/docker-registry --add --type=secret --secret-name=new-tls-ca-bundle -m /etc/pki/tls/certs