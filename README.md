# SSL Certificate Ansible Role
Ansible role to generate, sign and store SSL keys and certificates.

This role requires a cfssl 1.3+ installation with a config file stored in /etc/cfssl/config.json.
The jumperfly.cfssl role can be used for this if required but is not declared as a dependency to avoid repeated re-runs each time this role is used on the same host.

## Storage Backends
The role currently offers two mechanisms to store the generate the certificates using the ``ssl_cert_storage`` variable:
* file: store on local disk
* k8s-secret: store in Kubernetes as a tls secret

See the [Storage Configuration](#storage-configuration) section for more detail.

## Certificate authorities
The role currently offers two mechanisms to sign CSRs using the ``ssl_cert_ca_type`` variable:
* delegate: delegates to a CA host and uses cfssl on the delegate host
* acme-http: uses an acme CA host, by default Let's Encrypt

See the [CA Configuration](#ca-configuration) section for more detail.

## Base Configuration
| Key | Description |
|-----|-------------|
| ``ssl_cert_name``                | A unique name for the certificate. Used as a name when storing the certiificate. |
| ``ssl_cert_storage``             | Defines how to store the certificate and keys: one of file,k8s-secret. |
| ``ssl_cert_ca_type``             | Defines the CA used to sign the certificates: one of delegate,acme-http. |
| ``ssl_cert_type``                | Defines the cfssl profile to use: one of server,client,peer,intermediate. |
| ``ssl_cert_subject_common_name`` | The common name in the certificate subject, defaults to ``ssl_cert_name``. |
| ``ssl_cert_hosts``               | List (YAML) of hosts to be included as IP/DNS SAN entries. Defaults to the subject common name. |

## Subject Configuration
| Key | Description |
|-----|-------------|
| ``ssl_cert_subject_organization_name``        | Organisation. |
| ``ssl_cert_subject_country``                  | Country. |
| ``ssl_cert_subject_city``                     | City. |
| ``ssl_cert_subject_state``                    | State. |
| ``ssl_cert_subject_organizational_unit_name`` | Organisational unit. |

## Storage Configuration
### File Storage
This is the default storage mechanism.  Certificates are stored on the local file system.

| Key | Description |
|-----|-------------|
| ``ssl_cert_file_base_dir`` | The base directory used to store keys and certificates. |

### Kubernetes Secret Storage
Stores the key and certificate as a Kubernetes secret.  Reads a local kubeconfig file to locate the apiserver.
The configured ``ssl_cert_name`` is used as the name of the secret.
Three items are stored in the secret:
* ``tls.crt``: the certificate (required by the tls secret type)
* ``tls.key``: the private key (required by the tls secret type)
* ``ca.crt``: The CA certificate chain (not required by the tls secret type, but needed for some deployments)

| Key | Description |
|-----|-------------|
| ``ssl_cert_k8s_secret_namespace`` | The Kubernetes namespace to store the secret. |

## CA Configuration
### Delegate CA
This is the default CA to use for signing. An Ansible host is configured as a delegate.
The delegate must have cfssl and required intermediate/root certificates installed on the local file system.
Typcially the ``ssl_cert`` or ``root_ca`` roles are used to configure these on the delegate.

| Key | Description |
|-----|-------------|
| ``ssl_cert_ca_delegate``          | The name of the Ansible host to use as the delegate. |
| ``ssl_cert_ca_delegate_name``     | The name of the key/cert storage on the delegate host used to locate the certificate files. |
| ``ssl_cert_ca_delegate_base_dir`` | The base directory on the delegate holding the key/cert used for signing. Three files are expected, {name}-key.pem, {name}.pem and {name}-chain.pem. |

### ACME HTTP CA
Uses the Automatic Certificate Management Environment (ACME) protocol to sign certificates. HTTP validation is used, only a single host is currently supported.
A local httpd server is used to perform the HTTP validation. It is assume the ACME host can reach this server using the configurated host in the certificate.
Default configuration is to use Let's Encrypt Staging to sign the certificate.
The production directory, or an alternative ACME host should be configured to generate valid certificates.

| Key | Description |
|-----|-------------|
| ``ssl_cert_acme_account_key``              | The location of the ACME account key |
| ``ssl_cert_acme_autogenerate_account_key`` | If set to 'yes' a key is generated if not found. If set to 'no' the task will fail. |
| ``ssl_cert_acme_staging``                  | Determines whether to use the 'staging' or 'production' directory.  Defaults to staging. |
| ``ssl_cert_acme_staging_directory``        | The URL of the staging directory. |
| ``ssl_cert_acme_prod_directory``           | The URL of the production directory. |
| ``ssl_cert_acme_agreement``                | The URL of the ACME agreement. |

