# microxchg2017
Resources for my presentation at microXchg 2017 in Berlin, Germany.

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

# Links

## Spring Cloud Services for Pivotal Cloud Foundry
http://docs.pivotal.io/spring-cloud-services/

## Nimbus JOSE+JWT
https://connect2id.com/products/nimbus-jose-jwt

## Here are some excellent blog posts by Prabath Siriwardena, who explains the fundamentals very well.

[Securing Microservices](https://medium.facilelogin.com/securing-microservices-with-oauth-2-0-jwt-and-xacml-d03770a9a838#.pdhie0o6l)

[JWT, JWS and JWE for Not So Dummies!](https://medium.facilelogin.com/jwt-jws-and-jwe-for-not-so-dummies-b63310d201a3#.wac92a69y)
