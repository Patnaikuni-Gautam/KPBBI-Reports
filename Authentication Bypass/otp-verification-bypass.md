# OTP Verification Bypass via Response Manipulation

## Affected URL

[https://taskbear.com.au/](https://taskbear.com.au/).

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

### Original Workflow (Valid OTP Verification)

**Screenshot 1: Entering a valid OTP**
A valid 6-digit verification code (`252129`) is submitted for the test email `lunartermite80114@patchsec.site`.

<img width="907" height="753" alt="screenshot 1" src="https://github.com/user-attachments/assets/83cf0436-06fe-494f-8b3c-f790af360c8e" />


---

**Screenshot 2: Intercepting the valid OTP request**
The `POST /api/v1/uwa/auth/otp/verify` request containing the valid OTP is intercepted in Burp Suite, and response interception is enabled.

<img width="853" height="790" alt="screenshot 2" src="https://github.com/user-attachments/assets/5b85bb89-2d18-4b09-86d5-68ee62451d8f" />

---

**Screenshot 3: Capturing the successful verification response**
The server responds with `HTTP/2 201 Created` and a JSON payload containing the `contact` email and a `verifiedAt` timestamp.

<img width="1007" height="653" alt="screenshot 3" src="https://github.com/user-attachments/assets/a816c238-47cd-4757-94e3-1c8413acd049" />

---

### Bypass Workflow (Response Manipulation)

**Screenshot 4: Submitting an invalid OTP**
Using a new test email (`kindlemur12608@patchsec.site`), an arbitrary invalid OTP (`123456`) is submitted and the request is intercepted.

<img width="1917" height="761" alt="screenshot 4" src="https://github.com/user-attachments/assets/e52d4a71-4de5-43e3-8881-8add277f5c05" />

---

**Screenshot 5: Original response for the invalid OTP**
The server rejects the invalid code, returning `HTTP/2 401 Unauthorized` with the message `"OTP invalid or expired"`.

<img width="1918" height="820" alt="screenshot 5" src="https://github.com/user-attachments/assets/118cf121-6aa2-4a11-91fb-633ae141e7c4" />

---

**Screenshot 6: Modifying the HTTP response in Burp Suite**
Before forwarding to the client, the response status is changed to `HTTP/2 201 Created` and the body is replaced with the valid JSON structure (`contact` and `verifiedAt` fields) observed in Screenshot 3.

<img width="1908" height="781" alt="screenshot 6" src="https://github.com/user-attachments/assets/4f92ae48-84d6-4506-b2c7-c81b59b9c243" />

---

**Screenshot 7: Email verification bypassed**
Upon forwarding the modified response, the application displays `"Email verified successfully!"` and advances to the profile creation step.

<img width="879" height="908" alt="screenshot 7" src="https://github.com/user-attachments/assets/9040e8f5-7fef-4c39-89be-e7980f08265d" />

---

**Screenshot 8: Account creation completed**
After submitting the registration details, the account is created and the application redirects to the authenticated user dashboard.

<img width="1763" height="889" alt="screenshot 8" src="https://github.com/user-attachments/assets/a55dc487-378d-444f-af6f-7ccbc7615bfa" />

---

**Screenshot 9: Unverified email active in Account Security**
Navigating to the Account Security settings confirms that the account is active and associated with the unverified email address (`kindlemur12608@patchsec.site`).

<img width="1758" height="652" alt="screenshot 9" src="https://github.com/user-attachments/assets/9b46cc4c-acd5-4096-b059-c9c07726a3e7" />

## Impact

Potential security implications include:

1. **OTP verification bypass:** The intended OTP verification flow can be circumvented at the application interface level.
2. **Unverified account registration:** Registration appears successfully without valid OTP verification.
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
