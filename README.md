# EndtoEnd-DevOps
             DEVELOPER
                 |
                 |
             GITHUB
                 |
              Webhook
                 |
                 |
              JENKINS
                 |
          Checkout Source Code
                 |
                 |
           Install Node.js
                 |
                 |
           npm install / ci
                 |
                 |
            npm run build
                 |
                 |
            Docker Build
                 |
                 |
             AWS ECR
                 |
                 |
              AWS EC2
                 |
              Docker Pull
                 |
              Docker Run
                 |
                 |
             NGINX SERVER
                 |
                 |
          REACT APPLICATION
                 |
                 |
           PUBLIC IP:80
                |
                Whenever you push new code into GitHub, Jenkins will automatically build and deploy the latest version
