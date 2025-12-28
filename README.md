# Campus Event Management System

## 🎯 Project Overview

Welcome to the Campus Event Management System - a modern, cloud-native web application designed to revolutionize how students discover and engage with campus events. What makes this project special isn't just the application itself, but the sophisticated DevOps pipeline that powers it. Imagine pushing your code changes to GitHub and having them automatically deployed to a production-ready AWS infrastructure within minutes - no manual intervention, no complex deployment scripts, just pure automation magic.

This project serves as a comprehensive demonstration of modern cloud-native development practices, showcasing how enterprise-grade infrastructure can be managed entirely through code. When you commit changes to the main branch, an orchestrated chain of events begins: GitHub Actions springs into action, provisioning all necessary AWS resources if they don't exist, building optimized Docker containers for both frontend and backend, pushing them to Amazon's Elastic Container Registry, and finally deploying them to ECS Fargate clusters behind a load balancer. The entire journey from code commit to live production takes approximately 5-7 minutes max, resulting in a highly available, scalable application that's ready to handle real-world traffic.

## 🎯 What This Application Does

At its core, this is a full-featured event management platform tailored for university campuses. Students can seamlessly browse through a curated list of upcoming events, each with detailed information including descriptions, dates, locations, and available spots. The application provides an intuitive filtering system allowing users to narrow down events by categories such as Workshops, Hackathons, Tech Talks, Career Fairs, and Study Groups.

The registration system, while simulated using browser's local storage for demonstration purposes, showcases production-ready UI patterns including:

- **Real-time feedback** with loading states during operations
- **Instant updates** showing current registration counts
- **Optimistic UI updates** for smooth user experience
- **Error handling** with graceful fallbacks and retry mechanisms
- **State persistence** across browser sessions

![Uploading {2767F699-8429-4775-A5C0-9271E8901A4B}.png…]()


The frontend is built with React 18 and styled beautifully with Tailwind CSS, offering a responsive design that works flawlessly across desktop and mobile devices. The backend, powered by Node.js and Express, provides robust REST API endpoints with proper CORS configuration, health check endpoints for container orchestration, and structured logging for debugging and monitoring.

## 🚀 Quick Start Guide

Here's everything you need to do to have a production-ready application running on AWS infrastructure in under 15 minutes:

### Prerequisites
- **An AWS Account** with appropriate permissions (you'll need access to create ECS, VPC, ALB, ECR, and IAM resources)
- **GitHub Account** for hosting your repository
- **AWS Access Keys** (Access Key ID and Secret Access Key) with programmatic access

### Deployment Steps

**1. Clone and Prepare Your Repository**
```bash
git clone <this-repository-url>
cd End-to-End-Deployment-Automation-to-AWS-using-GitHub-Actions
```

**2. Create Your Own GitHub Repository**

Push this code to your own GitHub repository where you have administrative access to manage secrets.

**3. Configure AWS Credentials in GitHub Secrets**

Navigate to your GitHub repository settings, then to "Secrets and variables" → "Actions", and add the following secrets:
- `AWS_ACCESS_KEY_ID` - Your AWS access key
- `AWS_SECRET_ACCESS_KEY` - Your AWS secret key

**4. Trigger the Deployment**

Simply push any change to the main branch, or manually trigger the workflow:
```bash
git push origin main
```

That's genuinely all there is to it! GitHub Actions will orchestrate the entire deployment process automatically. You can monitor the progress in the "Actions" tab of your GitHub repository, where you'll see detailed logs of every step - from infrastructure provisioning to Docker image building, and finally the ECS service deployment.

### 🧹 Cleanup Resources

When you're done experimenting or need to delete all created AWS resources to avoid ongoing charges, you can do using GitHub Actions Workflow**
```bash
gh workflow run cleanup.yml
```

This requires the GitHub CLI (`gh`) to be installed and authenticated.

Both methods will systematically delete all created AWS resources including ECS services, clusters, load balancers, target groups, VPC components, security groups, ECR repositories (and all images), CloudWatch log groups, and IAM roles. The process typically completes in 3-5 minutes and leaves your AWS account clean.

## ⚙️ How the Deployment Pipeline Works

The beauty of this project lies in its sophisticated yet fully automated deployment pipeline. Every component works in harmony to deliver a seamless experience from code commit to production deployment. Here's the fascinating journey your code takes:

### Infrastructure Provisioning (First Deployment Only)

During the initial deployment, the GitHub Actions workflow intelligently checks for existing infrastructure. If none is found, it automatically provisions a complete, production-ready AWS environment:

- **VPC Setup**: Creates a Virtual Private Cloud with CIDR block 10.0.0.0/16, providing isolated network space
- **Network Architecture**: Configures two public subnets across different availability zones for high availability
- **Internet Connectivity**: Sets up an Internet Gateway and route tables enabling outbound internet access
- **Security Groups**: Establishes three distinct security groups with least-privilege access patterns:
  - ALB security group (allows HTTP traffic from anywhere)
  - Backend security group (allows traffic only from ALB on port 3000)
  - Frontend security group (allows traffic only from ALB on port 80)
- **Load Balancer**: Deploys an Application Load Balancer with health checks and target groups for both services
- **Container Registry**: Creates ECR repositories for storing Docker images
- **Logging Infrastructure**: Sets up CloudWatch log groups for centralized logging
- **IAM Roles**: Configures necessary execution roles with appropriate permissions

### Continuous Deployment Workflow (Every Push)

Once infrastructure exists, every subsequent push to the main branch triggers an optimized deployment:

1. **Source Checkout**: GitHub Actions checks out your latest code
2. **AWS Authentication**: Securely authenticates using your stored credentials
3. **Docker Build Process**: 
   - Multi-stage builds optimize image sizes
   - Frontend builds static assets with Vite before packaging with Nginx
   - Backend bundles Node.js application with all dependencies
4. **Image Registry**: Pushes tagged Docker images to ECR repositories
5. **Task Definition Updates**: Registers new ECS task definitions with latest image URIs
6. **Zero-Downtime Deployment**: 
   - ECS services are updated with force-new-deployment flag
   - Old tasks continue serving traffic during new task startup
   - Load balancer health checks ensure new tasks are ready
   - Only then does traffic shift to new containers
7. **Health Verification**: Waits for services to reach stable state with all tasks running
8. **URL Output**: Provides the live application URLs in the workflow logs

The entire pipeline is idempotent, meaning you can run it repeatedly without issues, and it's intelligent enough to create resources only when needed while updating existing ones gracefully.

## 🛠️ Technology Stack

This project leverages modern, industry-standard technologies across the entire development and deployment lifecycle:

### Frontend Technologies
- **React 18** - Utilizing the latest features including concurrent rendering and automatic batching for optimal performance
- **Vite** - Lightning-fast build tool providing instant hot module replacement during development and optimized production builds
- **Tailwind CSS** - Utility-first CSS framework enabling rapid UI development with consistent design patterns
- **React Router** - Client-side routing for seamless single-page application navigation experience
- **Nginx** - High-performance web server serving static assets in production with optimized caching and compression

### Backend Technologies
- **Node.js** - JavaScript runtime providing non-blocking I/O for scalable server-side applications
- **Express.js** - Minimalist web framework offering robust routing, middleware support, and RESTful API patterns
- **CORS** - Properly configured cross-origin resource sharing for secure frontend-backend communication
- **Health Check Endpoints** - Built-in endpoints for container orchestration and load balancer health monitoring

### Cloud Infrastructure & DevOps
- **AWS ECS Fargate** - Serverless container orchestration eliminating server management overhead
- **Application Load Balancer** - Layer 7 load balancing with path-based routing, health checks, and SSL/TLS termination capability
- **Amazon ECR** - Private Docker container registry with vulnerability scanning and lifecycle policies
- **Amazon VPC** - Isolated network environment with public subnets and custom route tables
- **CloudWatch Logs** - Centralized logging solution for debugging and monitoring application behavior
- **AWS IAM** - Fine-grained access control with task execution roles following least-privilege principles
- **GitHub Actions** - CI/CD platform orchestrating the entire build, test, and deployment pipeline
- **Docker** - Containerization platform ensuring consistent environments from development to production

### Key Architectural Decisions

- **Fargate over EC2**: Chose serverless containers to eliminate infrastructure management and optimize costs
- **Multi-stage Docker builds**: Reduces final image sizes by 60-70%, improving deployment speed and security
- **Path-based routing**: ALB routes `/api/*` to backend and `/*` to frontend, enabling single domain architecture
- **Security groups over NACLs**: Stateful firewall rules provide simpler yet robust security boundaries
- **Infrastructure as Code**: All AWS resources created via AWS CLI in GitHub Actions for reproducibility and version control

## Project Structure

```
campus-events-app/
├── .github/
│   └── workflows/
│       └── deploy.yml          # Complete CI/CD pipeline
├── backend/
│   ├── src/
│   │   ├── index.js            # Express server with API routes
│   │   └── routes/
│   │       └── events.js       # Event management endpoints
│   ├── Dockerfile              # Backend container definition
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── EventCard.jsx   # Individual event display with register/unregister
│   │   │   └── Navbar.jsx      # Navigation component
│   │   ├── pages/
│   │   │   ├── EventsPage.jsx  # Main events listing with filters
│   │   │   ├── AboutPage.jsx
│   │   │   ├── ContactPage.jsx
│   │   │   └── AccountPage.jsx
│   │   ├── App.jsx             # Main app component with routing
│   │   ├── main.jsx            # App entry point
│   │   └── index.css           # Global styles
│   ├── Dockerfile              # Frontend container definition
│   ├── nginx.conf              # Nginx production config
│   └── package.json
├── scripts/
│   ├── cleanup.sh              # Delete all AWS resources
│   └── secrets.sh              # Automate setting GitHub secrets from .env file
└── README.md
```

## 💻 Local Development Setup

Before deploying to AWS, you might want to run and test the application locally. Here's how to get both the frontend and backend running on your development machine:

### Backend Development Server

Navigate to the backend directory and start the Express server:

```bash
cd backend
npm install
npm run dev
```

The backend API will start on `http://localhost:3000`. You'll see colorful console output indicating successful startup, along with the available endpoints. The development server includes hot-reloading through nodemon, so any changes you make to the source files will automatically restart the server.

**Available Endpoints:**
- `GET /api/events` - Retrieves all campus events
- `GET /api/health` - Health check endpoint (returns server status)

### Frontend Development Server

In a separate terminal window, navigate to the frontend directory:

```bash
cd frontend
npm install
npm run dev
```

Vite will start the development server on `http://localhost:5173` (or the next available port if 5173 is occupied). The development experience includes:
- **Instant hot module replacement** - See your changes reflected immediately without full page reloads
- **Optimized dependency pre-bundling** - Fast cold start times even with large dependency trees
- **Rich error overlays** - Helpful error messages displayed directly in the browser

The frontend is pre-configured to proxy API requests to `http://localhost:3000`, so everything works seamlessly in development mode just as it would in production.

### Development Tips

- **Parallel Terminal Windows**: Keep both servers running in separate terminal windows for the best development experience
- **Browser DevTools**: Use React DevTools extension to inspect component state and props
- **API Testing**: Use tools like Postman or curl to test backend endpoints independently
- **Tailwind IntelliSense**: Install the official Tailwind CSS IntelliSense extension in VS Code for class name autocompletion

## 📋 Detailed Deployment Workflow Breakdown

The GitHub Actions workflow located at `.github/workflows/deploy.yml` orchestrates a comprehensive deployment process. Understanding each step helps you troubleshoot issues and customize the deployment for your specific needs:

### Phase 1: Infrastructure Foundation
- **VPC Creation** - Establishes isolated network space (10.0.0.0/16 CIDR)
- **Subnet Provisioning** - Two public subnets across different AZs (10.0.1.0/24 and 10.0.2.0/24)
- **Gateway Configuration** - Internet Gateway with route table associations for internet connectivity
- **Route Tables** - Custom routing enabling internet access from public subnets

### Phase 2: Security & Access Control
- **Security Group Setup** - Three-tier security model:
  - ALB Security Group (port 80, 3000 from internet)
  - Backend Security Group (port 3000 from ALB only)
  - Frontend Security Group (port 80 from ALB only)
- **IAM Role Creation** - ECS task execution role with ECR pull and CloudWatch logs permissions

### Phase 3: Load Balancing Configuration
- **ALB Deployment** - Application Load Balancer spanning both availability zones
- **Target Groups** - Separate target groups for frontend (port 80) and backend (port 3000)
- **Listener Rules** - HTTP listeners with health check paths configured
- **Health Checks** - Automated monitoring with 30-second intervals and 2-minute timeout thresholds

### Phase 4: Container Registry & Build
- **ECR Repositories** - Creates private repositories with image scanning enabled
- **Docker Image Building** - Multi-stage builds optimizing final image sizes
- **Image Tagging Strategy** - Uses Git commit SHA for traceability plus 'latest' tag
- **Registry Push** - Secure push to ECR with automatic authentication

### Phase 5: Container Orchestration
- **ECS Cluster Creation** - Fargate-enabled cluster for serverless container execution
- **CloudWatch Log Groups** - Centralized logging with 7-day retention policy
- **Task Definitions** - Specifies container configurations, resource requirements, and environment variables
- **Task Resources** - 512 CPU units and 1024MB memory per task (adjustable based on needs)

### Phase 6: Service Deployment & Stabilization
- **Service Creation/Update** - Deploys containers with desired count of 1 (horizontally scalable)
- **Load Balancer Integration** - Automatic registration of tasks with target groups
- **Rolling Updates** - Zero-downtime deployment with health check validation
- **Service Stabilization** - Waits for all tasks to reach RUNNING state with HEALTHY load balancer status
- **URL Output** - Displays live application URLs in workflow logs

### Monitoring Your Deployment

After triggering the workflow, you can monitor progress through:
- GitHub Actions tab showing real-time logs
- AWS ECS console displaying service status and task health
- CloudWatch Logs containing application logs from both frontend and backend containers
- ALB target health in EC2 console

The entire deployment process typically completes in 10-15 minutes for the first run, and 5-8 minutes for subsequent deployments as infrastructure already exists.

## ✨ Application Features

This event management platform comes packed with features that demonstrate modern web development best practices:

### Core Functionality
- **📅 Event Discovery & Browsing** - Comprehensive list of campus events with rich details including title, description, date, time, location, category, and available spots
- **🏷️ Advanced Category Filtering** - Intuitive filter system supporting multiple categories:
  - 🛠️ Workshops - Hands-on learning sessions
  - 💻 Hackathons - Coding competitions and buildathons
  - 🎤 Tech Talks - Speaker sessions and presentations
  - 💼 Career Fairs - Networking and recruitment events
  - 📚 Study Groups - Collaborative learning sessions
- **✅ Event Registration System** - One-click registration/unregistration with instant feedback (demonstrates production-ready patterns using browser localStorage)
- **🎯 Registered Events View** - Dedicated filter to display only events you've signed up for, making it easy to track your commitments

### User Experience Enhancements
- **📱 Fully Responsive Design** - Seamlessly adapts to any screen size, from mobile phones to large desktop monitors
- **⚡ Real-time UI Updates** - Registration counts and button states update instantly without page refreshes
- **🔄 Loading States** - Professional loading indicators during asynchronous operations providing clear feedback
- **❌ Robust Error Handling** - Graceful error messages with user-friendly explanations and retry options when operations fail
- **🎨 Modern UI Components** - Clean, professional interface built with Tailwind CSS featuring cards, gradients, and smooth animations
- **🔍 Visual Feedback** - Interactive elements respond to hover, click, and focus states for enhanced usability

### Technical Features
- **Client-side Routing** - Fast navigation between pages without full page reloads using React Router
- **State Management** - Efficient local state management with React hooks for optimal performance
- **API Integration** - RESTful API communication with proper error handling and retry logic
- **Data Persistence** - Browser localStorage maintains registration state across sessions
- **Optimistic Updates** - UI updates immediately while background operations complete for snappy user experience

## 🎓 Learning Outcomes & Skills Demonstrated

This project serves as a comprehensive portfolio piece showcasing multiple in-demand technical skills that are highly valued in the software industry:

### DevOps & Cloud Engineering
- **Infrastructure as Code (IaC)** - Complete AWS infrastructure provisioned through declarative code rather than manual console clicking, enabling reproducibility and version control
- **CI/CD Pipeline Design** - End-to-end automation from code commit to production deployment with zero manual intervention
- **Container Orchestration** - Hands-on experience with ECS Fargate, understanding task definitions, service management, and cluster operations
- **Cloud Networking** - Deep dive into VPC architecture, subnet design, routing tables, internet gateways, and security group configurations
- **Load Balancing** - Practical implementation of Application Load Balancer with target groups, health checks, and path-based routing
- **Security Best Practices** - Implementing least-privilege access with IAM roles, security groups, and secrets management through GitHub Actions

### Software Development
- **Full-stack Development** - Building complete applications with React frontend and Node.js backend, understanding how they communicate
- **RESTful API Design** - Creating well-structured API endpoints with proper HTTP methods, status codes, and error handling
- **Modern JavaScript** - Using ES6+ features, async/await patterns, and functional programming concepts
- **State Management** - Managing application state efficiently with React hooks and local storage persistence
- **Responsive Web Design** - Creating layouts that work beautifully across all device sizes using modern CSS frameworks

### Containerization & Deployment
- **Docker Mastery** - Writing optimized Dockerfiles with multi-stage builds reducing image sizes by 60-70%
- **Container Registry Management** - Working with private ECR repositories, image tagging strategies, and vulnerability scanning
- **Production Web Server Configuration** - Configuring Nginx for optimal static file serving with proper caching headers
- **Environment Configuration** - Managing different configurations for development, staging, and production environments

### Professional Development Practices
- **Git Workflow** - Utilizing version control effectively with meaningful commits and branch strategies
- **Documentation** - Writing clear, comprehensive documentation that helps others understand and use the project
- **Debugging & Monitoring** - Using CloudWatch Logs for application monitoring and troubleshooting production issues
- **Cost Optimization** - Understanding AWS pricing and implementing cleanup workflows to avoid unnecessary charges

### Real-World Problem Solving
- **High Availability Design** - Multi-AZ deployment ensuring application continues running even if one availability zone fails
- **Zero-Downtime Deployments** - Implementing rolling updates that deploy new versions without service interruption
- **Health Check Implementation** - Creating robust health check endpoints for automated monitoring and recovery
- **Error Resilience** - Building applications that gracefully handle failures and provide meaningful feedback to users

## 🚀 Customization & Extension Ideas

This project provides a solid foundation that you can extend and customize for your own learning:

- **Add Database Integration** - Integrate RDS PostgreSQL or DynamoDB for persistent data storage
- **Implement Authentication** - Add user login with AWS Cognito or Auth0
- **Email Notifications** - Use AWS SES to send event reminders and confirmation emails
- **Search Functionality** - Implement full-text search with OpenSearch or Algolia
- **Analytics Dashboard** - Track event popularity, user engagement, and registration trends
- **API Gateway** - Add API Gateway for rate limiting, API keys, and request throttling
- **CDN Integration** - Use CloudFront for global content delivery and improved performance
- **Automated Testing** - Add unit tests, integration tests, and end-to-end testing with Cypress
- **Multiple Environments** - Create separate staging and production environments
- **Blue-Green Deployments** - Implement advanced deployment strategies for even safer releases

## 🔧 Troubleshooting Common Issues

### Deployment Taking Too Long / Services Won't Stabilize

**Symptoms:** GitHub Actions workflow gets stuck at "Waiting for services to stabilize..." for more than 5 minutes.

**Common Causes & Solutions:**

1. **ECR Connection Issues**
   - **Error:** `ResourceInitializationError: unable to pull secrets or registry auth`
   - **Cause:** Tasks can't reach Amazon ECR to pull Docker images
   - **Solution:** Verify security groups allow outbound traffic (they should by default). Run cleanup and redeploy:
     ```bash
     ./scripts/cleanup.sh
     git push origin main
     ```

2. **Health Check Failures**
   - **Error:** Tasks keep restarting, never become healthy
   - **Cause:** Application not responding on correct port or health check path
   - **Solution:** 
     - Verify backend responds at `/health` endpoint
     - Check CloudWatch Logs for application errors
     - Ensure container port matches task definition (3000 for backend, 80 for frontend)

3. **Insufficient Resources**
   - **Cause:** Tasks fail due to memory/CPU limits
   - **Solution:** Check task stopped reason in ECS console, increase CPU/memory in task definitions if needed

### Checking Deployment Status

View detailed logs in AWS Console:

```bash
# Check ECS Service Events
aws ecs describe-services --cluster campus-events-cluster \
  --services campus-events-backend campus-events-frontend \
  --query 'services[*].events[0:5]'

# Check Task Status
aws ecs list-tasks --cluster campus-events-cluster \
  --service-name campus-events-backend

# View Container Logs (replace TASK_ID)
aws logs tail /ecs/campus-events-backend --follow
```

### Target Groups Show Unhealthy

**Check target health:**
```bash
aws elbv2 describe-target-health \
  --target-group-arn <your-target-group-arn>
```

**Common issues:**
- Security groups blocking ALB → container traffic
- Application not listening on 0.0.0.0 (listening on localhost only)
- Wrong health check path configured

### Application Not Accessible via ALB

**Wait Time:** Allow 1-2 minutes after deployment for DNS propagation and health checks to pass.

**Verify ALB is working:**
```bash
# Get ALB DNS
aws elbv2 describe-load-balancers \
  --names campus-events-alb \
  --query 'LoadBalancers[0].DNSName' --output text

# Test health endpoint
curl http://<alb-dns>:3000/health
```

### Clean Slate Approach

If deployment continues to fail, the safest approach is to clean up all resources and start fresh:

```bash
# Run cleanup
./scripts/cleanup.sh

# Wait 2-3 minutes for deletion to complete

# Trigger fresh deployment
git push origin main
```

This ensures no misconfigured resources are interfering with the deployment.

## 🤝 Contributing & Feedback

This project is open for contributions, suggestions, and improvements! Whether you're a beginner looking to learn or an experienced developer wanting to add features:

- **🐛 Found a Bug?** - Open an issue with detailed reproduction steps
- **💡 Have an Idea?** - Suggest new features or improvements through GitHub issues
- **🔧 Want to Contribute?** - Fork the repository, make your changes, and submit a pull request
- **⭐ Found it Helpful?** - Give it a star on GitHub to show your support!

## 📝 License & Usage

Feel free to use this project as a learning resource, portfolio piece, or starting point for your own applications. Fork it, modify it, and make it your own. The best way to learn is by doing, so don't hesitate to experiment and break things - that's how we grow as developers!

---

**Built with ❤️ for the developer community by Muna Bhattarai | Happy Coding! 🚀**



