# newarch
New architecture to replace a monolithic backend

# Problem Summary
## Scaling of certain functionalities at peak load is challenging
There are multiple domains within the business that are involved in the complete system, including but not limited to 
* product catalog management
* price management
* order management
* inventory management

Specific functionalities around product catalog management have scaling issues.

There are various geographies - regulatory regions within the business that need to be catered to, namely
* Australia
* Canada
* India

They have their own regulatory requirements, consumer behavior and cultural conventions.

## Individual business domains are interdependent and need simultaneous upgrades
The backend is a monolith ie everything ships together, it is not possible to upgrade the system corresponding to one domain area alone, everything must be upgraded together. This increases time to deploy and reduces nimbleness.
## Inability to use different vendors / technologies per domain
At some point it may have been prudent to use one vendor for everything but it is also a liability because the organization has to be live with that vendor's strengths and weaknesses, and the vendor has leverage over the organization.

# Functional Requirements
## Product Catalog Service
* add product
* retrieve product list based on search criteria eg product type
* remove product

## Pricing Service
* get price of a given product, and also other details of the product from Product Catalog Service.

# Non functional Requirements
* Product Catalog Service and Pricing Service should be upgradable / maintainable independent of each other even though Pricing Service may be a client for Product Catalog Service.
* json over http using REST 
* any technology stack is fine - but must be justified, same for databases
* hosting per service must be independent of each other
* load balancing
* service discovery
* monitoring
* migration from current monolith to independent services must cause minimal disruption to business.
* system should be highly available, minimal downtime for upgrades

# Proposed Solution
The proposed API documentation is available as listed below. 
## Product Catalog API
[Catalog API Documentation](http://htmlpreview.github.io/?https://github.com/alkuma/newarch/blob/master/products-api.html)
## Pricing API
[Pricing API Documentation](http://htmlpreview.github.io/?https://github.com/alkuma/newarch/blob/master/pricing-api.html)

