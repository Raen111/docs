---
title: About dependency review
intro: 'Dependency review lets you catch insecure dependencies before you introduce them to your environment, and provides information on license, dependents, and age of dependencies.'
product: '{% data reusables.gated-features.dependency-review %}'
shortTitle: Dependency review
versions:
  fpt: '*'
  ghes: '>= 3.2'
  ghae: '*'
  ghec: '*'
type: overview
topics:
  - Advanced Security
  - Dependency review
  - Vulnerabilities
  - Dependencies
  - Pull requests
redirect_from:
  - /code-security/supply-chain-security/about-dependency-review
---

## About dependency review

{% data reusables.dependency-review.feature-overview %}

If a pull request targets your repository's default branch and contains changes to package manifests or lock files, you can display a dependency review to see what has changed. The dependency review includes details of changes to indirect dependencies in lock files, and it tells you if any of the added or updated dependencies contain known vulnerabilities.

Sometimes you might just want to update the version of one dependency in a manifest and generate a pull request. However, if the updated version of this direct dependency also has updated dependencies, your pull request may have more changes than you expected. The dependency review for each manifest and lock file provides an easy way to see what has changed, and whether any of the new dependency versions contain known vulnerabilities.

By checking the dependency reviews in a pull request, and changing any dependencies that are flagged as vulnerable, you can avoid vulnerabilities being added to your project. For more information about how dependency review works, see "[AUTOTITLE](/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/reviewing-dependency-changes-in-a-pull-request)."

For more information about configuring dependency review, see "[AUTOTITLE](/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-dependency-review)."

{% data variables.product.prodname_dependabot_alerts %} will find vulnerabilities that are already in your dependencies, but it's much better to avoid introducing potential problems than to fix problems at a later date. For more information about {% data variables.product.prodname_dependabot_alerts %}, see "[AUTOTITLE](/code-security/dependabot/dependabot-alerts/about-dependabot-alerts#dependabot-alerts-for-vulnerable-dependencies)."

Dependency review supports the same languages and package management ecosystems as the dependency graph. For more information, see "[AUTOTITLE](/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph#supported-package-ecosystems)."

For more information on supply chain features available on {% data variables.product.product_name %}, see "[AUTOTITLE](/code-security/supply-chain-security/understanding-your-software-supply-chain/about-supply-chain-security)."

{% ifversion ghec or ghes %}

## Enabling dependency review

The dependency review feature becomes available when you enable the dependency graph. For more information, see "{% ifversion ghec %}[Enabling the dependency graph](/code-security/supply-chain-security/understanding-your-software-supply-chain/about-the-dependency-graph#enabling-the-dependency-graph){% elsif ghes %}[Enabling the dependency graph for your enterprise](/admin/code-security/managing-supply-chain-security-for-your-enterprise/enabling-the-dependency-graph-for-your-enterprise){% endif %}."
{% endif %}

{% ifversion fpt or ghec or ghes or ghae > 3.5 %}

## Dependency review enforcement

The action is available for all {% ifversion fpt or ghec %}public repositories, as well as private {% endif %}repositories that have {% data variables.product.prodname_GH_advanced_security %} enabled.

{% data reusables.dependency-review.action-enterprise %}

You can use the {% data variables.dependency-review.action_name %} in your repository to enforce dependency reviews on your pull requests. The action scans for vulnerable versions of dependencies introduced by package version changes in pull requests, and warns you about the associated security vulnerabilities. This gives you better visibility of what's changing in a pull request, and helps prevent vulnerabilities being added to your repository. For more information, see [`dependency-review-action`](https://github.com/actions/dependency-review-action).

![Screenshot of a workflow run that uses the Dependency review action.](/assets/images/help/graphs/dependency-review-action.png)

By default, the {% data variables.dependency-review.action_name %} check will fail if it discovers any vulnerable packages. A failed check blocks a pull request from being merged when the repository owner requires the dependency review check to pass. For more information, see "[AUTOTITLE](/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-status-checks-before-merging)."

{% ifversion fpt or ghec or ghes %}The action uses the dependency review REST API to get the diff of dependency changes between the base commit and head commit. You can use the dependency review API to get the diff of dependency changes, including vulnerability data, between any two commits on a repository. For more information, see "[AUTOTITLE](/rest/dependency-graph#dependency-review)."{% endif %}{% ifversion dependency-review-submission-api %} The action also considers dependencies submitted via the dependency submission API. For more information about the dependency submission API, see "[AUTOTITLE](/code-security/supply-chain-security/understanding-your-software-supply-chain/using-the-dependency-submission-api)."

{% data reusables.dependency-review.works-with-submission-api-beta %}

{% endif %}

{% ifversion dependency-review-action-configuration %}
You can configure the {% data variables.dependency-review.action_name %} to better suit your needs. For example, you can specify the severity level that will make the action fail{% ifversion dependency-review-action-licenses %}, or set an allow or deny list for licenses to scan{% endif %}. For more information, see "[AUTOTITLE](/code-security/supply-chain-security/understanding-your-software-supply-chain/configuring-dependency-review#configuring-the-dependency-review-github-action)."
{% endif %}

{% endif %}

{% ifversion dependency-review-submission-api %}

## Best practices for using the dependency review API and the dependency submission API together

The dependency review API and the {% data variables.dependency-review.action_name %} both work by comparing dependency changes in a pull request with the state of your dependencies in the head commit of your target branch, which is usually your default branch.

If your repository only depends on statically defined dependencies in one of {% data variables.product.prodname_dotcom %}’s supported ecosystems, the dependency review API and the {% data variables.dependency-review.action_name %} work consistently.

However, you may want your dependencies to be scanned during a build and then uploaded to the dependency submission API. In this case, there are some best practices you should follow to ensure that you don’t introduce a race condition when running the processes for the dependency review API and the dependency submission API, since it could result in missing data.

The best practices you should take will depend on whether you use {% data variables.product.prodname_actions %} to access the dependency submission API and the dependency review API, or whether you use direct API access.

### Using GitHub Actions to access the dependency submission API and the dependency review API

If you use {% data variables.product.prodname_actions %} to access the dependency submission API or the dependency review API:
   * Make sure you run all of your dependency submission actions in the same {% data variables.product.prodname_actions %} workflow as your {% data variables.dependency-review.action_name %}. This will give you control over the order of execution, and it will ensure that dependency review will always work.
   * If you do choose to run the {% data variables.dependency-review.action_name %} separately, for instance, as a required workflow, you should:
     + Set `retry-on-snapshot-warnings` to `true`.
     + Set `retry-on-snapshot-warnings-timeout` to slightly exceed the typical run time (in seconds) of your longest-running dependency submission action.

### Using direct API access to the dependency submission API and the dependency review API

If you don’t use {% data variables.product.prodname_actions %}, and your code relies on direct access to the dependency submission API and the dependency review API:
   * Make sure you run the code that calls the dependency submission API first, and then run the code that calls the dependency review API afterwards.
   * If you do choose to run the code for the dependency submission API and the dependency review API in parallel, you should implement a retry logic and note the following:
     + When there are snapshots missing for either side of the comparison, you will see an explanation for that in the `x-github-dependency-graph-snapshot-warnings` header (as a base64-encoded string). Therefore, if the header is non-empty, you should consider retrying.
     + Implement a retry logic with exponential backoff retries.
     + Implement a reasonable number of retries to account for the typical runtime of your dependency submission code.
{% endif %}
