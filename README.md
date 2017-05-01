# microxchg2017
Resources for "Beyond OAuth2: End to End Microservice Security" presented at microXchg 2017 in Berlin, Germany.

# Video
https://www.youtube.com/watch?v=G7A6ftCbVQY

# Slides
https://docs.google.com/presentation/d/1gmMlvBW8JNGGo0rY_CnMt6qRYGCGVfQCvevkxVYhXWs/edit?usp=sharing

# Feedback
Whether we see more of this solution or not entirely depends on feedback from the community. Please direct any feedback, both about the presentation, the ideas, or the solution, through github issues on the repo linked below. You can also email me.

# The Code
https://github.com/william-tran/microservice-security-jose

# Questions After the Talk

## Isn't having to specify a policy against an entire call stack breaking encapsulation? It feels against the spirit of microservices.

In my demo I was very specific in the policies, and I neglected to show you the case where "Other than needing to know that the request came through the front door 'Shop', I don't care." Here's such a policy:

```yaml
suppliers:
  policy:
    for-each-of-the-following:
    - enforce-the-first-matching-rule:
      - tokens-that:
          have-operation:
            equal-to: resupply
        must:
          come-from:
          - any-number-of-apps: true
          - app-name:
              equal-to: shop
```
The idea here is that you should be able to specify a policy that enforces only the things you really care about. 

## Can I use something other than UAA as the Authorization Server (AS)?

Absolutely. The most important property that underpins this whole scheme actually lies in your edge application. Let's consider two scenarios, the first one where the client is a web application where the token is stored in server side session, as I describe in my talk. The second scenario, one that I didn't illustrate in my talk, is where the client is a mobile app or browser based JS app. In this second scenario the client is a non-confidential client, and the user's token can be used to invoke the edge service directly.

In the first scenario, the browser can invoke the web application through an authenticated session, where the session identifier (eg JSESSIONID cookie) can be seen as a sort of "token". This JSESSIONID never gets propagated downstream. The token from the AS is stored in the server side session and contains the claims statelessly eg as a JWT, or statefully as an opaque token. Whether the AS token is a JWT or not doesn't matter, what does matter is that it cannot be used on its own to invoke any service, either downstream, or to be replayed back onto the web application. This can't be allowed because this token is supposed to be reused, but only as a way to determine what the AS says the user is allowed to do. Downstream services cannot take the AS's token and replay it, on it's own, to any other component. 

In the second scenario, a non-confidential client (eg mobile app or SPA) gets a token from the AS and invokes a service with it. Because the service can be invoked with this token, it cannot propagate it downstream, otherwise downstream services can use it to re-invoke that service. The user's claims still need to be propagated however, because downstream services need to know things about the user. In this case, the service at the edge can either generate and sign its own assertion about the user, or exchange the AS token it received for an assertion signed by the AS. In either case, this user assertion cannot be used to invoke any service on its own, or be used to derive a token that can be used to invoke any service directly. 

This is the situation I talk about in https://github.com/william-tran/microservice-security-jose/issues/1. In the descriptions above, UAA doesn't give you anything special over any other AS that helps with this. You just need to be careful about how services can be invoked, and that the thing (eg token, session ID) used to invoke a service, either at your edge or anywhere downstream, is never propagated.

## How does the authorization server and the service registry authenticate clients? Can we use client certificates?

In UAA, clients can authenticate with the UAA via client_id, and in the case of confidential clients, a client_secret, via HTTP Basic. Client registration in UAA is done by an actor (user or system) with the permission to register clients, the registration contains the  client_id+client_secret. Services must authenticate with the service registry to register their public key in a trusted way. In [Spring Cloud Services](http://docs.pivotal.io/spring-cloud-services/1-3/common/security-overview.html) (SCS), services authenticate with Eureka using an OAuth token obtained from UAA via client credentials grant. This is the value add that SCS provides on top of the OSS Eureka server. I mention in my talk other service registries like Zookeper and Consul that include their own authentication mechanisms / plugins out of the box, but I personally haven't used these. 

I've also seen [Google Service Accounts](https://developers.google.com/identity/protocols/OAuth2ServiceAccount) use self signed JWTs to authenticate clients, instead of client_id+client_secret. The act of registering a client in Google is done by the Devloper and they get back a private key for their app to create self signed JWTs. 

You can definitely use client certificates to authenticate clients instead of OAuth tokens (and transitively, client_id+client_secret), as used in PCF and SCS. If using mutual TLS, you might not need to do the JWS wrapping step, as mutual TLS provides the non-resuable, not propagated authentication; the authorization in the form of the nested JWT tokens remains the same though.  

## Can we use a full PKI instead of the lightweight service registry? Or can I combine them?

Yes, if you have a PKI you can use that to provide the key material for signing. For verification, certificates can be imbedded in the JWS header itself and in that way the JWS is truly self-verifying (as long as you trust the thing that signed the certificate; that trust anchor is provided by your PKI). Embedding the certificate in the JWS header via x5c (https://tools.ietf.org/html/rfc7515#section-4.1.6) is pretty heavy size-wise, so the service registry could be used as a way for the recipient to look up the full certificate given an identifier in the JWS header like the "kid", and then cache this locally.

## What happens if the central service registry is compromised? 

If you are using the registry on its own without a PKI, then the entire system would be compromised. It's as if you were using a PKI and the private key for your CA were compromised.  

# Links

## Spring Cloud Services for Pivotal Cloud Foundry
http://docs.pivotal.io/spring-cloud-services/

## Nimbus JOSE+JWT
https://connect2id.com/products/nimbus-jose-jwt

## Here are some excellent blog posts by Prabath Siriwardena, who explains the fundamentals very well.

[Securing Microservices](https://medium.facilelogin.com/securing-microservices-with-oauth-2-0-jwt-and-xacml-d03770a9a838#.pdhie0o6l)

[JWT, JWS and JWE for Not So Dummies!](https://medium.facilelogin.com/jwt-jws-and-jwe-for-not-so-dummies-b63310d201a3#.wac92a69y)
