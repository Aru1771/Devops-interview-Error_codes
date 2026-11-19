Learn these five especially well for DevOps interviews: 401, 403, 404, 502, 503, and 504.:


1. 401 -	Authentication, token, credentials
2. 403	IAM, RBAC, permissions, authorization
3. 404 -	URL, route, Service, resource
4. 502 -	Nginx, ALB, Ingress, upstream application
5. 503	- Pod health, readiness probes, endpoints
6. 504	- Application response time, network, timeout settings

HTTP 401 — Unauthorized:
------------------------

Meaning: The server cannot verify who you are.

Common reasons:

    - Invalid username or password.
    - Expired access token.
    - Missing authentication token.
    - Invalid API key.
DevOps example:

Your Jenkins pipeline calls an application API:
curl -H "Authorization: Bearer INVALID_TOKEN" \
https://api.example.com/payment


Response:
HTTP/1.1 401 Unauthorized


What to check:

- Is the token correct?
- Has the token expired?
- Is the Authorization header present?
- Are application credentials configured correctly?
Remember: 401 = Authentication failure.


HTTP 403 — Forbidden:
---------------------

Meaning: The server understands who you are, but you don't have permission to access the resource.

Common reasons:
- Insufficient IAM permissions.
- Kubernetes RBAC restrictions.
- Application access policies.
- S3 bucket policy denies access.
DevOps example:

Your EC2 instance accesses an S3 bucket, but its IAM role doesn't have s3:GetObject permission.

Response:
HTTP/1.1 403 Forbidden


What to check:
- IAM role and policies.
- S3 bucket policy.
- Kubernetes RBAC permissions.
- Application authorization rules.
Remember: 403 = Identity may be valid, but permission is denied.


HTTP 404 — Not Found:
---------------------

Meaning: The requested resource or URL cannot be found.

Common reasons:
- Incorrect URL.
- API endpoint doesn't exist.
- Kubernetes Ingress path is incorrect.
- Application route is missing.
DevOps example:

Your application is running, but the user accesses:
https://example.com/payments


The application only supports:
https://example.com/payment


Response:
HTTP/1.1 404 Not Found


What to check:
- URL and API path.
- Nginx location configuration.
- Ingress routing rules.
- Application routes.
- Whether the requested resource exists.
Remember: 404 = Requested resource not found.


HTTP 502 — Bad Gateway:
------------------------

Meaning: A gateway or proxy received an invalid response from the backend server.

Meaning: A gateway or proxy received an invalid response from the backend server.

Common reasons:
- Backend application crashed.
- Nginx cannot communicate properly with the application.
- ALB receives an invalid response from its target.
- Incorrect upstream configuration.
DevOps example:

Your architecture:
User
  |
  v
AWS ALB
  |
  v
Nginx
  |
  v
Payment application


Payment application crashes or returns an invalid response to Nginx.

The gateway may return:
HTTP/1.1 502 Bad Gateway


What to check:
- Nginx error logs.
- ALB target health.
- Backend application logs.
- Backend port and connectivity.
- Upstream configuration.
Remember: 502 = Gateway received a bad response from upstream.



HTTP 503 — Service Unavailable:
-------------------------------

Service unavailable
Meaning: The server is temporarily unable to handle the request.

Common reasons:
- Application is overloaded.
- All backend Pods are unhealthy.
- No ready endpoints behind a Kubernetes Service.
- Maintenance or temporary service shutdown.
DevOps example:

Your Kubernetes application has three Pods:
payment-service
      |
      +---- Pod 1 (NotReady)
      +---- Pod 2 (NotReady)
      +---- Pod 3 (NotReady)


There are no ready backend Pods.

The ingress controller or load balancer may return:
HTTP/1.1 503 Service Unavailable


What to check:
kubectl get pods
kubectl get svc
kubectl get endpointslices
kubectl describe pod payment-pod


Check readiness probes, application health and resource utilization.

Remember: 503 = Service temporarily unavailable.


HTTP 504 — Gateway Timeout:
---------------------------

Meaning: A gateway or proxy waited for the backend, but the backend did not respond within the allowed time.

Common reasons:
- Slow database queries.
- Application taking too long.
- Network connectivity problems.
- Upstream timeout configuration is too low.
DevOps example:
User
  |
  v
AWS ALB
  |
  v
Payment application
  |
  v
Database


The database query takes 90 seconds, but the ALB's idle timeout is reached first.

The user receives:
HTTP/1.1 504 Gateway Timeout


What to check:
- Application response time.
- Database query performance.
- ALB idle timeout.
- Nginx proxy timeout.
- Network connectivity.
Remember: 504 = Backend didn't respond in time.


One important distinction: 502, 503 and 504 do not always mean the application itself is faulty. The load balancer, proxy, networking layer or an upstream dependency may be responsible.


<img width="750" height="297" alt="image" src="https://github.com/user-attachments/assets/f0797c9c-1bed-4d6d-adfe-4826178bfd41" />

