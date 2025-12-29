# Complete Guide to Setting Up E-commerce for Sherab with WordPress/WooCommerce

## What Is This Integration?

-   WordPress website as your course catalog and storefront
-   WooCommerce handling payments, orders, and customer management
-   Automatic enrollment in Sherab when courses are purchased
-   Professional e-commerce features like coupons, analytics, and
    marketing tools

## How It Works

1.  **Installation Process**

    ``` bash
    pip install -U tutor-contrib-wordpress
    tutor plugins enable wordpress
    tutor dev launch
    ```

    This automatically installs:

    -   Complete WordPress installation
    -   WooCommerce e-commerce plugin
    -   Sherab Commerce plugin (the bridge between systems)

    You do NOT need to:

    -   Install WooCommerce separately
    -   Download the OpenEdX Commerce plugin manually
    -   Set up the traditional OpenEdX e-commerce service
    -   Run database migrations

2.  **Manual Configuration Required**

    -   Create OAuth2 application in Sherab Django admin
    -   Configure WordPress plugin settings with OAuth2 credentials
    -   Test the connection between systems

3.  **You can retrieve these configuration values by running:**

    ``` bash
       tutor wordpress config printroot    
    ```
    
## Detailed Setup Steps

### Phase 1: OAuth2 Setup in Sherab

-   Access Django admin at `http://localhost:8000/admin`
-   Go to **OAuth2 Provider → Applications → Add Application**
-   Configure:
    -   Client type: Confidential
    -   Authorization grant type: Client credentials
    -   User: Select staff user
    -   Name: "WordPress Commerce"
-   Copy Client ID and Secret

### Phase 2: WordPress Configuration

-   Access WordPress at `http://localhost:8080`
-   In admin → WooCommerce → Settings → Integration → OpenEdX
-   Enter:
    -   LMS Domain
    -   Client ID
    -   Client Secret
-   Test connection with "Generate JWT Token"

### Phase 3: Creating Course Products

-   In WordPress admin → Products → Add New
-   Add name, price, description, images
-   Check **OpenEdX Course**
-   Configure:
    -   Course ID (e.g., `course-v1:MySchool+Math101+2024`)
    -   Course Mode (audit, honor, verified, etc.)
-   Publish

## Understanding Course Modes

  --------------------------------------------------------------------------------
  Mode                 Description                   Certificate   Payment
                                                                   Required
  -------------------- ----------------------------- ------------- ---------------
  audit                Free access, no certificate   No            No

  honor                Free access with certificate  Yes           No

  verified             Paid with verified            Yes           Yes
                       certificate                                 

  no-id-professional   Professional, no ID           Yes           Yes
                       verification                                
  --------------------------------------------------------------------------------

## Purchase-to-Enrollment Workflow

1.  Purchase the course in WordPress (Sherab-Edustore)
2.  Order processed by WooCommerce
3.  OpenEdx Plugin detects order
4.  Enrollment request created with billing email
5.  API call to OpenEdX
6.  Enrollment created in specified course/mode

## Refund Handling

-   Refund in WooCommerce → Plugin soft unenrolls user
-   User loses course access but retains progress

## Course Discovery and Pricing

-   Prices and buy buttons only on WordPress storefront
-   OpenEdX pages: add manual "Buy this course" links to WordPress

## Student Experience

-   Browse courses in WordPress
-   Purchase via WooCommerce
-   Automatic enrollment in OpenEdX
-   Learn in OpenEdX

## System Architecture

-   WordPress: storefront, payments, accounts, marketing
-   OpenEdX: course content, tracking, certificates, forums
-   Integration plugin: enrollment, refunds, error handling

## Technical Communication

-   OAuth2 client credentials flow
-   REST API calls WordPress → OpenEdX
-   JWT tokens
-   Webhook-style triggers on order completion

## Production Considerations

-   Set up Stripe/PayPal gateways
-   Configure SSL
-   Enable email notifications
-   Backups and monitoring
-   Load balancing for high traffic

## Common Issues

-   **JWT Token Fails:** check domain, client ID/secret, staff user
-   **Enrollments Not Working:** confirm Course ID, mode, and email
    match
-   **Access Issues:** verify user email and course availability

## References

-   [WordPress OpenEdX Commerce
    Plugin](https://wordpress.org/plugins/openedx-commerce/)
-   [OpenEdX API Connection
    Decision](https://docs.openedx.org/projects/wordpress-ecommerce-plugin/en/latest/decisions/0002-api-connection.html)
-   [Certificates
    Configuration](https://public.docs.edunext.co/en/latest/external/course_creators/authoring_courses/certificates-configuration.html)
-   [Enrollment
    Tracks](https://public.docs.edunext.co/en/latest/external/course_creators/prepare_test_launch_courses/configure-enrollment-tracks.html)
