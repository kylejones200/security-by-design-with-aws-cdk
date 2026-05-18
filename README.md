# Security by design with AWS CDK

Published: 2024-10-18
Medium: [https://medium.com/@kyle-t-jones/security-by-design-with-aws-cdk-5187da3a2c74](https://medium.com/@kyle-t-jones/security-by-design-with-aws-cdk-5187da3a2c74)

## Business context

Security is job zero when building any application. AWS CDK makes it easier for developers to incorporate security best practices while building scalable systems. AWS offers various services and tools to enhance security, including IAM roles, encryption, logging, monitoring, and compliance tools. These tools reduce the risk of misconfigurations and security issues when using CDK.

Security must be at the forefront of every resource definition and interaction when working with CDK. AWS CDK enables secure defaults for AWS resources while allowing developers to enforce strict security policies programmatically. This includes enforcing encryption, setting appropriate permissions, using the principle of least privilege, and ensuring that access control is granular and explicit.

Many AWS resources have security features that must be configured explicitly by default. AWS CDK provides high-level constructs (L2 constructs) that often come with secure defaults, such as enabling encryption for S3 buckets or enforcing HTTPS-only communication for API Gateway. While these defaults are a good starting point, you can further tighten the security posture by customizing them.

## About

Place the code for this article in this repository.
The original article export is saved as `article.md`.

## Files

Add your `.ipynb`, `.py`, `.yaml`, `.js`, `.ts`, or other project files here.

## Disclaimer

Educational/demo code only. Not financial, safety, or engineering advice. Use at your own risk. Verify results independently before any production or operational use.

## License

MIT — see [LICENSE](LICENSE).