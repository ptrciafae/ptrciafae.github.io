+++
date = '2025-06-21T16:59:42+08:00'
draft = false
tag = ["programming"]
category = ["blog"]
title = '001 Setting Up a Custom Domain for Your Hugo Blog Hosted on GitHub Pages'
+++

It’s been a month since I got laid off—and what a ride it’s been.

After returning from a soul-stirring pilgrimage (more on that in a future post), I’m finally kicking off this blog to document the deep dives, detours, and rabbit holes I find myself exploring as I learn, unlearn, and rebuild.

For a first post, I'll detail how I setup this blog.

# Setting Up a Custom Domain for Your Blog Hosted on GitHub Pages

Using [GitHub Pages](https://pages.github.com/), [Namecheap](https://namecheap.com/), [Route 53](https://aws.amazon.com/route53/), and [Terraform](https://developer.hashicorp.com/terraform)

## Background

I chose these technologies for their familiarity and developer-friendliness. Hugo uses Markdown — a format developers regularly use: from writing technical documentation to pull request descriptions. Deploying via GitHub Actions to GitHub Pages is second nature to most developers, myself included. It just made sense.

Adding a custom domain gives the blog a more professional look — the foundation of a personal brand. It's cleaner, easier to remember, and shareable. More importantly, it sets the stage for growth. I plan to spin up other sites and services under the ptrciafae.dev domain as this ecosystem evolves.

I went with Namecheap for domain registration based on strong community recommendations (mostly on Reddit). While its UI feels dated, it works reliably — and that’s good enough.

For DNS, I opted to use AWS Route 53. It integrates seamlessly with the rest of the AWS ecosystem, which I intend to leverage for future projects. It also gave me a great excuse to finally get hands-on with Terraform.

## Prerequisites

Make sure you have the following before you start:

- An [AWS account](https://aws.amazon.com/)
- [Terraform CLI](https://developer.hashicorp.com/terraform/downloads) installed and configured with AWS credentials
- A [Hugo site already deployed via GitHub Pages](https://gohugo.io/host-and-deploy/host-on-github-pages/)

## 1. Buy a Domain on Namecheap

I registered the domain `ptrciafae.dev` through Namecheap, which is often recommended for its reliability and customer experience.

💡 Cost: A two-year registration without any add-ons was USD $27.32.

No DNS configuration is needed on Namecheap beyond setting the nameservers to AWS. We'll handle everything else from Route 53.

## 2. Set Up a Hosted Zone in Route 53 with Terraform

Create a Route 53 hosted zone to manage your domain’s DNS records through AWS.

Terraform snippet:

```
resource "aws_route53_zone" "ptrciafae_hosted_zone" {
  name = "ptrciafae.dev"
}
```

📝 This defines a hosted zone for your root domain. Once applied, Route 53 will provide nameservers that you'll copy to Namecheap.

## 3. Point Domain to Route 53

After creating the hosted zone, go to your domain settings in Namecheap and update the Custom DNS Nameservers to match the ones provided by Route 53 (e.g., `ns-xxx.awsdns-xx.net`).

DNS changes may take several hours to propagate globally.

## 4. Configure GitHub Pages for Custom Domain

On your Hugo site's GitHub repository, navigate to:

⚙️ _Settings_ → _Pages_ → _Custom domain_

Enter your subdomain, e.g., `blog.ptrciafae.dev`.

GitHub Pages will attempt to verify and provision an SSL certificate, so make sure the DNS is correctly set up.

## 5. Add CNAME Record in Route 53 with Terraform

To map `blog.ptrciafae.dev` to your GitHub Pages site, add a CNAME record pointing to `ptrciafae.github.io`.

Terraform snippet:

```
resource "aws_route53_record" "github_pages_cname" {
  zone_id = aws_route53_zone.ptrciafae_hosted_zone.zone_id
  name    = "blog.ptrciafae.dev"
  type    = "CNAME"
  ttl     = 300
  records = ["ptrciafae.github.io"]
}
```

## 5. Add CNAME Record in Route 53 with Terraform

To map `blog.ptrciafae.dev` to your GitHub Pages site, add a CNAME record pointing to `ptrciafae.github.io`.

Terraform snippet:

```
resource "aws_route53_record" "github_pages_cname" {
  zone_id = aws_route53_zone.ptrciafae_hosted_zone.zone_id
  name    = "blog.ptrciafae.dev"
  type    = "CNAME"
  ttl     = 300
  records = ["ptrciafae.github.io"]
}
```

## 6. Add a CNAME File to Your GitHub Repo

Create a CNAME file in the root of your Hugo project. This file should contain just your custom domain name, without extra spaces:

```
blog.ptrciafae.dev
```

This ensures GitHub Pages knows what domain to bind to your site during deployment.
