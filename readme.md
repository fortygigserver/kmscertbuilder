# certbuilder

A Python library for creating and signing X.509 certificates.

 - [Related Crypto Libraries](#related-crypto-libraries)
 - [Current Release](#current-release)
 - [Dependencies](#dependencies)
 - [Installation](#installation)
 - [License](#license)


## Related Crypto Libraries

*kmscertbuilder* is a fork of certbuilder, and is a Python library for constructing X.509 certificates using KMS Keys. It
provides a high-level interface with knowledge of RFC 5280 to produce, valid,
correct certificates without terrible APIs or hunting through RFCs.

*certbuilder* is part of the modularcrypto family of Python packages:

 - [asn1crypto](https://github.com/wbond/asn1crypto)
 - [oscrypto](https://github.com/wbond/oscrypto)
 - [csrbuilder](https://github.com/wbond/csrbuilder)
 - [certbuilder](https://github.com/wbond/certbuilder)
 - [crlbuilder](https://github.com/wbond/crlbuilder)
 - [ocspbuilder](https://github.com/wbond/ocspbuilder)
 - [certvalidator](https://github.com/wbond/certvalidator)

## Current Release

0.14.2 - [changelog](changelog.md)

## Dependencies

 - [*asn1crypto*](https://github.com/wbond/asn1crypto)
 - [*oscrypto*](https://github.com/wbond/oscrypto)
 - Python 2.6, 2.7, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8, 3.9 or pypy

## Basic Usage

The example shows how to create a self-signed certificate using a KMS Key pair by passing in a dictionary of name information to
the `KMSCertificateBuilder()`. This generated self-signed certificate (root CA) can be used with `KMSCertificateSigner` to sign end-entity certificate signing requests (CSRs).constructor:

```bash
git clone https://github.com/fortygigserver/kmscertbuilder
pip3 install -r ./kmscertbuilder/requires/requirements.txt
```

``` python 
import sys
sys.path.append('./kmscertbuilder')

from asn1crypto import x509, pem, csr
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
    certification_request = csr.CertificationRequest.load(pem.unarmor(f.read())[2])


end_entity_certificate = KMSCertificateSigner(certification_request, kms_root_ca, kms_arn)

with open('/path/to/my/env/CustomerHsmCertificate.crt', 'wb') as f:
    f.write(pem_armor_certificate(end_entity_certificate))
```

## License

*kmscertbuilder* is licensed under the terms of the MIT license. See the
[LICENSE](LICENSE) file for the exact license text.

