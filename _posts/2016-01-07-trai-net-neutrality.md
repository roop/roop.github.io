---
layout: post
title: "Responding to TRAI on net neutrality in India"
description: "These are the comments on net neutrality that I submitted to the Telecom Regulatory Authority of India"
category: posts
---

The Telecom Regulatory Authority of India (TRAI) has invited comments
from the public on allowing "Differential Pricing for Data Services".

The [consultation paper](http://www.trai.gov.in/Content/ConDis/20761_0.aspx) they are inviting comments on isn't very long or hard to understand, so I read through that today and sent them my answers to the questions they are seeking answers for. (It also happens that as of now, today is the last date for submissions.)

This is the response I sent, with the questions included inline:

---

Thank you for inviting the general public into the discussion by
inviting comments on this issue. I appreciate it greatly and it
improves the confidence I have on the governing process in my country.

On to the questions now:

> Question 1: Should the TSPs (Telecom Service Providers / Internet
> Service Providers) be allowed to have differential pricing
> for data usage for accessing different websites, applications or
> platforms?

Yes, but only under certain conditions (explained below).

> Question 2: If differential pricing for data usage is permitted, what
> measures should be adopted to ensure that the principles of
> non-discrimination, transparency, affordable internet access,
> competition and market entry and innovation are addressed?

To ensure that competition among e-commerce and internet-based
businesses remains fair, ANY ONE of the following should be enforced:

<ol><li>
    <p>The power of deciding what websites / applications / platforms are
    available for differential pricing through a TSP should NOT reside
    with a commercial for-profit entity</p>
</li></ol>

<p>(OR)</p>

<ol start="2"><li>
    <p>Websites / applications / platforms of e-commerce and
    internet-related businesses should NOT be allowed to be accessed
    with differential pricing</p>
</li></ol>

Additionally, to ensure the security and privacy of consumers using the
differential pricing scheme, the following should be enforced:

 1. HTTPS proxying should maintain end-to-end encryption, so that secure
    requests are not "peeked into" by the TSP or any other intermediary

There are multiple ways in which the above conditions can be
implemented. Two possible schemes are illustrated below:

 1. Scheme #1: Distributed decision making
 
    The decision on what websites / applications / platforms are
    available for differential access price should not be made by a
    centralized entity - that choice is solely for the website /
    application / platform to make.

    For example, an alternative to FreeBasics can be implemented as
    follows:
    
    FreeBasics has some technical criteria that need to be fulfilled for
    the website to be proxied through Facebook's proxy server. Given
    this scenario, the consumer can ask for *any* website (say
    http://example.com/page) from his mobile device to the proxy server;
    the proxy server shall find out from example.com whether it wants to
    make /page available through FreeBasics using a pre-arranged method
    (e.g. just like robots.txt, use a freebasics-access.txt); if yes,
    the proxy server can proceed to fetch the page, and if the page
    passes the required technical criteria, the proxy can forward the
    page to the consumer's device. All negotiations happen automatically
    and transparently - the decision of whether a website is available
    with zero-rating shall reside with the website owner, and not with
    the TSP or Facebook. All HTTPS traffic is routed through the proxy
    as raw packets (like a HTTP CONNECT tunnel), without decryption by
    the proxy or any other intermediary.

 2. Scheme #2: Only government, non-profit and news websites

    E-commerce websites shall not be allowed to be offered under
    differential pricing. Only websites owned by governmental,
    non-profit and news organizations (e.g. Wikipedia, MAMA, Aaj Tak,
    IRCTC, BESCOM) shall be allowed. A commercial entity (e.g. the TSP
    or Facebook) can decide whether an applying website should be
    included under the differential pricing scheme or not. However, a
    government entity (say TRAI) can oversee the process, enforce that
    only government / news / non-profit websites are included, and
    arbitrate any disputes that may arise.

> Question 3. Are there alternative methods/technologies/business
> models, other than differentiated tariff plans, available to achieve
> the objective of providing free internet access to the consumers? If
> yes, please suggest/describe these methods/technologies/business
> models. Also, describe the potential benefits and disadvantages
> associated with such methods/technologies/business models?

If we exclude differentiated pricing, providing free internet access
means that the consumer is allowed to access the whole of the internet
without differentiating on what website s/he's visiting.

The cost of that internet access can be funded in many ways, some of
which are:

 - The consumer obtains free data access upto to a certain limit along
   with the purchase of a mobile device, plan or service
 - The consumer obtains free data access upto to a certain limit on
   watching video ads (on unbilled data) for a stipulated time
 - The consumer obtains free data access upto to a certain limit as a
   loyalty benefit

The advantage with these schemes is that they can provide free internet
access to consumers without affecting fairness of competition among
e-commerce and internet-based businesses.

However, even with these schemes, it is still essential to make sure
that the security / privacy aspects (see my answer to Question #2 above)
are considered.

> Question-4: Is there any other issue that should be considered in the
> present consultation on differential pricing for data services?

No.

---


