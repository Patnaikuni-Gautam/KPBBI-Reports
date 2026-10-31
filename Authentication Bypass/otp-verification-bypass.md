# OTP Verification Bypass via Response Manipulation

## Affected URL

Anonymized — target URL omitted.

## Description

During the assessment of an email-based OTP verification mechanism, testing was performed to determine whether the verification process could be bypassed through HTTP response manipulation using Burp Suite.

A valid OTP response and an invalid OTP response were captured and compared. The response associated with the invalid OTP was then modified to resemble the successful verification response.

After forwarding the modified response, the application proceeded to the dashboard despite the invalid OTP submission.

This behavior indicates a potential weakness in the application's OTP verification flow, particularly its handling of client-visible HTTP responses.

## Steps to Reproduce

1. Initiate registration using a controlled test email address.
2. Intercept a valid OTP submission using Burp Suite Proxy and capture the successful response.
3. Repeat registration using another controlled test email address.
4. Submit an invalid OTP and capture the rejection response.
5. Compare the successful and unsuccessful responses.
6. Modify the invalid OTP response to resemble the successful response.
7. Forward the modified response.
8. Observe that the application proceeds to the dashboard despite the invalid OTP.

## POC

**Screenshot 1: Successful OTP verification response**

[Insert Screenshot 1 Here]

---

**Screenshot 2: Invalid OTP response**

[Insert Screenshot 2 Here]

---

**Screenshot 3: Modified response using Burp Suite**

[Insert Screenshot 3 Here]

---

**Screenshot 4: Dashboard displayed after response manipulation**

[Insert Screenshot 4 Here]

## Impact

Potential security implications include:

1. **OTP verification bypass:** The intended OTP verification flow can be circumvented at the application interface level.
2. **Unverified account registration:** Registration may appear successful without valid OTP verification.
3. **Potential unauthorized access:** If the backend accepts the resulting verification state, protected functionality may become accessible without successful OTP validation.
4. **Circumvention of security controls:** Reliance on client-side response handling may allow invalid verification states to be treated as successful.

## Remediation

1. Enforce OTP validation exclusively on the server side.
2. Never rely on client-side response handling to determine verification status.
3. Validate account verification state on every protected API endpoint.
4. Create authenticated sessions only after successful server-side OTP verification.
5. Enforce OTP expiration and single-use restrictions.
6. Implement rate limiting for repeated OTP verification attempts.
7. Test invalid, expired, replayed, and manipulated OTP scenarios.

## Conclusion

The assessment demonstrated that modifying the HTTP response associated with an invalid OTP caused the application to proceed to the dashboard.

This indicates a weakness in the OTP verification flow. Further server-side validation is required to establish whether the behavior also results in unauthorized access to protected functionality.
