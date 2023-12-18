# kmscertbuilder Documentation

*kmscertbuilder* is a fork of certbuilder, and is a Python library for constructing X.509 certificates using KMS Keys. It
provides a high-level interface with knowledge of RFC 5280 to produce, valid,
correct certificates without terrible APIs or hunting through RFCs.

Since its only dependencies are the
[*asn1crypto*](https://github.com/wbond/asn1crypto#readme) and
[*oscrypto*](https://github.com/wbond/oscrypto#readme) libraries, it is
easy to install and use on Windows, OS X, Linux and the BSDs.

The documentation consists of the following topics:

 - [Basic Usage](#basic-usage)

## Basic Usage

The example shows how to create a self-signed certificate using a KMS Key pair by passing in a dictionary of name information to
the `KMSCertificateBuilder()`. This generated self-signed certificate (root CA) can be used with `KMSCertificateSigner` to sign end-entity certificate signing requests (CSRs).constructor:

```bash
git clone https://github.com/fortygigserver/kmscertbuilder
```

``` python 
import sys
sys.path.append('.\kmscertbuilder')

from asn1crypto import x509, pem
from kmscertbuilder import KMSCertificateBuilder, KMSCertificateSigner, pem_armor_certificate


kms_arn = 'arn:aws:kms:eu-west-1:xxxxxxxxxxxx:key/1234abcd-12ab-34cd-56ef-1234567890ab'

builder = KMSCertificateBuilder(
    {
        'country_name': 'IE',
        'state_or_province_name': 'Meath',
        'locality_name': 'East Meath',
        'organization_name': 'Palmep Tech',
        'common_name': 'Patrick',
    },
    kms_arn
)
builder.self_signed = True
builder.ca = True
root_kms_ca = builder.build(kms_arn)

with open('/path/to/my/env/root_kms_ca.crt', 'wb') as f:
    f.write(pem_armor_certificate(root_kms_ca))


# Read end-entity certificate and sign with root

with open('/path/to/my/env/ClusterCsr.csr', 'rb') as f:
    certification_request = x509.TbsCertificate.load(pem.unarmor(f.read())[2])


end_entity_certificate = KMSCertificateSigner(certification_request, kms_root_ca, kms_arn)

with open('/path/to/my/env/CustomerHsmCertificate.crt', 'wb') as f:
    f.write(pem_armor_certificate(end_entity_certificate))
```

All name components must be unicode strings. Common name keys include:

 - `country_name`
 - `state_or_province_name`
 - `locality_name`
 - `organization_name`
 - `common_name`

Less common keys include:

 - `organizational_unit_name`
 - `email_address`
 - `street_address`
 - `postal_code`
 - `business_category`
 - `incorporation_locality`
 - `incorporation_state_or_province`
 - `incorporation_country`
