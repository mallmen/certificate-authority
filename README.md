> This repository has been tested on MacOS Big Sur and Fedora 36.  It has been tested using Ansible 2.9.26 and requires the collection `community.crypto` be installed on your control node.

### Creating a Certificate Authority
Ansible playbook to create a self-signed certificate authority.  This will add the CA to the appropriate trust store for Mac and Fedora/RHEL.  

1. Configure `hosts` to run on the appropriate system in your environment

        sed -i 's/hosts: .*$/hosts: <your_target_host>/' generate_ca.yaml

1. Execute the playbook

    The following variables should be set in the playbook or passed at execution (requires sudo for adding CA to trust store):

        ca_path: <location for your certificate files> (must be set)
        filename: rootCA (default), change as desired (basename only)
        common_name: Root CA (default), set as desired for name of your CA
        passphrase: <password> (optional, commented out by default)

        ansible-playbook -i <inventory> generate_ca.yaml --ask-become-pass
    

### Creating a Signed Certificate
Ansible playbook to create a certificate using an existing local CA

1. Configure `hosts` to run on the appropriate system in your environment

        sed -i 's/hosts: .*$/hosts: <your_target_host>/' generate_certificate.yaml

1. Execute the playbook

    The following variables should be set in the playbook or passed at execution:

        ca_path: <location for your certificate files> (must be set)
        san_list: <comma separated list of hosts for certificate SAN> (must be set)
        subject: <first host in san_list> (default), customize as desired for certificate subject
        filename: <first host in san_list> (default), customize as desired (basename only)
        passphrase: <passphrase for CA key> (optional), commented by default

    The playbook will ensure that `san_list` is provided.  If generating a wildcard certificate, the `*` will be removed for the `subject` and `filename` variables if those are not specified specifically.

        ansible-playbook -i <inventory> generate_certificate.yaml -e san_list=host.example.com -e subject=HostCertificate

    > Although the `subject` should be able to have spaces, I encountered some issues when using spaces and a single host in the `san_list`.