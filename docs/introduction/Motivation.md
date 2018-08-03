# Motivation behind LambdaGrid

LambdaGrid exists because we think shipping a polished UI requires too much tedium. LambdaGrid abstracts away the tedium so that developers can configure their UI with small amounts of JavaScript.

All code in classic UI development, where you build the UI from scratch, falls under one of two categories:

1. Business and application logic
2. Infrastructure and maintenance

Developers can stick with the former while delegating the latter to LambdaGrid.

## The problem

UI is hard for a lot of reasons, from the tooling ecosystem, to the requisite skillset, to the available talent, to the inherent complexities of UI development. The net result is UI development requiring more time, money, and effort to ship a UI which, often, underwhelms the customer.

## The solution

With LambdaGrid, the whole UI already exists. A LambdaGrid UI is an ecosystem of packages that sit on top of the UI infrastructure. Developers express their business logic by writing small amounts of JavaScript for the packages they want to use.

LambdaGrid is not a framework in that it is not opinionated on how to build the UI. Rather, LambdaGrid packages can be composed in whatever way the developer wishes in order to run their UI. Architecting LambdaGrid to be the composition of packages rather than an opinionated framework is our way to ensure LambdaGrid will scale gracefully with the developer's business requirements.

## Inspirations

The main influence behind LambdaGrid's design is AWS. AWS exposes APIs with which developers can configure their services. Behind the APIs, AWS takes care of the infrastructure and maintenance underlaying their services. The result is that any developer can manage a sophisticated cloud computing platform. LambdaGrid aims to be AWS for UI by allowing any developer to manage a sophisticated UI.

Another way to think about LambdaGrid is "UI as a service." If there were a UI vendor who would take care of managing our UI for us, and we just configured the UI ourselves, then LambdaGrid is the vendor that we'd want for ourselves.
