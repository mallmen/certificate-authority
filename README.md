### Creating a Certificate Authority
Steps to create a self-signed cerficate authority.  You must configure your systems to trust the root certificate created by this process.

1. Create a directory to store your certificates

        mkdir ~/myca/{certs,private}

1. Create the key

        cd ~/myca/private
        openssl genrsa -des3 -out myCA.key 2048

1. Create the certificate

        cd ~/myca/certs
        openssl req -x509 -new -nodes -key ../private/myCA.key -sha256 -days 1825 -out myCA.pem

        Enter pass phrase for myCA.key:
        You are about to be asked to enter information that will be incorporated
        into your certificate request.
        What you are about to enter is what is called a Distinguished Name or a DN.
        There are quite a few fields but you can leave some blank
        For some fields there will be a default value,
        If you enter '.', the field will be left blank.
        -----
        Country Name (2 letter code) [AU]:US
        State or Province Name (full name) [Some-State]:Arizona
        Locality Name (eg, city) []:Gilbert
        Organization Name (eg, company) [Internet Widgits Pty Ltd]:
        Organizational Unit Name (eg, section) []:
        Common Name (e.g. server FQDN or YOUR name) []:
        Email Address []:mallmen@redhat.com

1. Add certificate to Macbook trust store

        sudo security add-trusted-cert -d -r trustRoot -k "/Library/Keychains/System.keychain" myCA.pem

1. Create a certificate

        openssl genrsa -out api.ocp4.example.com.key 2048

1. Generate a csr

        openssl req -new -key api.ocp4.example.com.key -out api.ocp4.example.com.csr

        You are about to be asked to enter information that will be incorporated
        into your certificate request.
        What you are about to enter is what is called a Distinguished Name or a DN.
        There are quite a few fields but you can leave some blank
        For some fields there will be a default value,
        If you enter '.', the field will be left blank.
        -----
        Country Name (2 letter code) []:US
        State or Province Name (full name) []:Arizona
        Locality Name (eg, city) []:Gilbert
        Organization Name (eg, company) []:
        Organizational Unit Name (eg, section) []:
        Common Name (eg, fully qualified host name) []:API Certificate ocp4.example.com cluster
        Email Address []:mallmen@redhat.com

        Please enter the following 'extra' attributes
        to be sent with your certificate request
        A challenge password []:

1. Create extentions file

        cat <<EOF > api.ocp4.example.com.ext
        authorityKeyIdentifier=keyid,issuer
        basicConstraints=CA:FALSE
        keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
        subjectAltName = @alt_names

        [alt_names]
        DNS.1 = api.ocp4.example.com
        EOF

1. Create the cert

        openssl x509 -req -in api.ocp4.example.com.csr -CA myCA.pem -CAkey ../private/myCA.key -CAcreateserial -out api.ocp4.example.com.crt -days 825 -sha256 -extfile api.ocp4.example.com.ext
