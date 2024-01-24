---
comments: True
layout: notebook
title: Deployment
description: An in depth deployment lesson
type: hacks
toc: True
courses: {'csa': {'week': 19}}
---

# Deployment Mini Guide

## Mini-Guide: Deploying Your Site with AWS

### Prerequisites
1. **GitHub Repository:**
   - Must have a backend repository on GitHub

3. **Domain Configuration:**
   - Have a configured Domain Name pointing to the Public IP of your deployment server using AWS Route 53

### AWS EC2 Access
1. **Login to AWS Console:**
   - Access AWS Management Console
   - Navigate to "EC2" and select "Instances."

2. **Instance Selection:**
   - Choose the appropriate instance (CSP or CSA) based on your project
   - <img width="700" src="https://github.com/The-GPT-Warriors/DeploymentLesson/assets/107821010/38f208bf-74d1-4d0f-b930-c3ddec75e4df">

3. **Terminal Access:**
   - Access the deployment server using either csp.nighthawkcodingsociety.com or csa.nighthawkcodingsociety.com
   - Enter the username and password

### Application Setup
1. **Finding Port:**
   - Run `docker ps` on AWS EC2 terminal to find an available port starting with 8—
   - This allows you to identify an available port for the application

2. **Docker Setup:**
   - Before configuring your Dockerfile, ensure that Docker is installed (for our class, it is already installed but it is good practice to verify that it is installed)
   - Open a terminal and run the following command to check the docker version: ``docker --version``
   - Verify the port you found using docker ps on AWS EC2 is matched in your Dockerfile and docker-compose.yml

> What your Dockerfile should look like

```# syntax=docker/dockerfile:1
FROM openjdk:18-alpine3.13
WORKDIR /app
RUN apk update && apk upgrade && \
    apk add --no-cache git 
COPY . /app
RUN ./mvnw package
CMD ["java", "-jar", "target/spring-0.0.1-SNAPSHOT.jar"]
EXPOSE 8---
```

> What your docker-compose.yml should look like

```version: '3'
services:
  web:
    image: your_image_name
    build: .
    ports:
      - "8---:8---"
    volumes:
       - ./volumes:/volumes
    restart: unless-stopped
```

   - Once the ports match, test your setup by running ``docker-compose up -d``

3. **Testing:**
   - Test the application locally before deployment
   - Run `docker-compose up` or `sudo docker-compose up` in VSCode terminal
   - Open `http://localhost:8---` in your browser to check if it runs smoothly

### Server Setup
1. **AWS EC2 Terminal:**
   - Setup the server environment and fetch the project code
   - Clone your backend repo: `git clone github.com/server/project.git your_repo`
   - Navigate to the repo: `cd your_repo`

2. **Build and Test:**
   - Build and test the application on the server
   - Build the site: `docker-compose up -d --build`
   - Test your site: `curl localhost:8---` (replace '8---' with your port)

### DNS & NGINX Setup
1. **Route 53 DNS:**
   - Configure DNS for domain mapping
   - Set up DNS subdomain for your backend server in AWS Route 53
   - Emaad will go over this in-depth later


2. **NGINX Configuration:**
   - Navigate to `/etc/nginx/sites-available` in the terminal
   - Create an NGINX config file and configure it accordingly
   - Configure NGINX as a reverse proxy for the application

3. **Validation and Restart:**
   - Validate with `sudo nginx -t`
   - Restart NGINX: `sudo systemctl restart nginx`
   - Test your domain on your desktop browser (http://)
   - These commands allow you to validate your NGINX configuration and restarts for changes to take effect

### Certbot Configuration
1. **Run Certbot:**
   - Configure SSL certificates for secure communication
   - Execute `sudo certbot --nginx` and follow prompts
   - Choose appropriate options for HTTPS activation

2. **Verify HTTPS:**
   - Test your domain in the browser using HTTPS, paste your link into the browser with HTTPS 

### Changing Code and Deployment Updates
1. **VSCode Changes:**
   - Before updating, run `git pull` to sync changes
   - Make code changes and test using Docker Desktop
   - This will synchronize your code changes that you have tested locally

2. **Deployment Update:**
   - If all goes well, sync changes via UI or `git push` from the terminal
   - This allows you to deploy updated code to the server


### Pulling Changes into AWS EC2
1. **AWS EC2 Terminal:**
   - Pull and update code on the server, then restart the application
   - Navigate to your repo: `cd ~/my_unique_name`
   - Stop the server: `docker-compose down`
   - Pull changes: `git pull`
   - Rebuild and start the container: `docker-compose up -d --build`

### Optional Troubleshooting Checks on AWS EC2
1. **Check Server Status:**
   - Check the status of your website
   - Use commands like `curl localhost:8---` and `docker-compose ps` for verification

# CORS
- Cross-Origin Resource Sharing (CORS) is a security feature implemented by web browsers to control how web pages in one domain can request and interact with resources from another domain.
[corserror](/images/cors.png)

- CORS is a set of rules that enable or restrict cross-origin (cross-site) HTTP requests made by scripts running on a web page. 
- It helps to prevent potentially harmful requests and enhances web security which is why it is so important

## Implementation on the Backend
1. MvcConfig.java


```python
package com.nighthawk.spring_portfolio;
import org.springframework.context.annotation.*;
import org.springframework.web.servlet.config.annotation.*;

@Configuration
public class MvcConfig implements WebMvcConfigurer {

    // This method sets up a custom index page for the "/login" path
    // Defines how the login page is accessed within app
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login").setViewName("login");
    }

    // Configures the location for uploaded files outside the app's resources
    // CRUCIAL for file upload functionality -> make sure files stored and can be accessed properly by frontend
    @Override
    public void addResourceHandlers(final ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/volumes/uploads/**").addResourceLocations("file:volumes/uploads/");
    }

    // Sets up CORS settings --> allows requests from specified origins 
    // This case is GitHub Pages site & local dev site
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**").allowedOrigins("https://nighthawkcoders.github.io", "http://localhost:4000");
    }
}
```

2. Securityconfig.java


```python
// Configure security settings for Spring app 

@Configuration
@EnableWebSecurity  // Enable basic Web security features
@EnableMethodSecurity(prePostEnabled = true)
public class SecurityConfig {

    @Autowired
	private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;

	@Autowired
	private JwtRequestFilter jwtRequestFilter;

	@Autowired
	private PersonDetailsService personDetailsService;

    // @Bean  // Sets up password encoding style
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

	// Configures the authentication manager to load user details 
	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(personDetailsService).passwordEncoder(passwordEncoder());
	}

	@Bean
	public AuthenticationManager authenticationManager(AuthenticationConfiguration authenticationConfiguration) throws Exception {
		return authenticationConfiguration.getAuthenticationManager();
	}
	
    // Configure security settings, including CORS 
		@Bean
		public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
			http
				.csrf(csrf -> csrf
					.disable()
				)
				// List the requests/endpoints that need to be authenticated
				.authorizeHttpRequests(auth -> auth
					.requestMatchers("/authenticate").permitAll()
					.requestMatchers("/mvc/person/update/**", "/mvc/person/delete/**").authenticated()
					.requestMatchers("/api/person/post/**", "/api/person/delete/**").authenticated()
					.requestMatchers("/**").permitAll()
				)
				// CORS support is enabled within the security configuration
				.cors(Customizer.withDefaults())
				// Set up specific headers related to CORS
				// Ensure that the necessary headers are included in the HTTP responses
				.headers(headers -> headers
					.addHeaderWriter(new StaticHeadersWriter("Access-Control-Allow-Credentials", "true"))
					.addHeaderWriter(new StaticHeadersWriter("Access-Control-Allow-ExposedHeaders", "*", "Authorization"))
					.addHeaderWriter(new StaticHeadersWriter("Access-Control-Allow-Headers", "Content-Type", "Authorization", "x-csrf-token"))
					.addHeaderWriter(new StaticHeadersWriter("Access-Control-Allow-MaxAge", "600"))
					.addHeaderWriter(new StaticHeadersWriter("Access-Control-Allow-Methods", "POST", "GET", "OPTIONS", "HEAD"))
					//.addHeaderWriter(new StaticHeadersWriter("Access-Control-Allow-Origin", "https://nighthawkcoders.github.io", "http://localhost:4000"))
				)
				.formLogin(form -> form 
					.loginPage("/login")
				)
				.logout(logout -> logout
					.logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
					.logoutSuccessUrl("/")
				)
				.exceptionHandling(exceptions -> exceptions
					.authenticationEntryPoint(jwtAuthenticationEntryPoint)
				)
				// Configures the session management to use a stateless approach--> Server does not store session information on the server-side between requests --> all the necessary information for authentication is contained within each request
				// Pro: more scalable, servers can handle requests independently 
				.sessionManagement(session -> session
					.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
				)
				// Add a filter to validate the tokens with every request
				.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
			return http.build();
	}
}

```

# DotEnv
- DotEnv is the use of a .env file within a project to handle sensitive data, including API keys and database credentials.
- The term "dotenv" is frequently associated with a specific library or tool that integrates these variables into the application's environment.
- Its core objective is to refrain from putting confidential information, like access tokens, directly into the source code or version control. Instead, such details are put in the .env file, customized for each distinct environment.

## DotEnv in relation to JWT
- JWTs are digitally signed using either a secret (HMAC) or a public/private key pair (RSA or ECDSA) which safeguards them from being modified by the client or an attacker
[jwt](/images/jwt.png)
- Using a .env file, you can store your JWT secret key and keep sensitive information secure. 

## Implementation
1. Navigate to your project's root directory using the command line: example - 
```cd /home/aliyatang/vscode/aliyaBlog```
2. Initialize a new package.json file for your project:
```npm init -y```
3. Install dotenv
```npm install dotenv```
4. Create .env file in root of project, in this file set JWT secret key: example - ```JWT_SECRET=your_secret_key```
5. Make a .js file, require and configure dotenv so you can load variables from .env file into process.env: ```require('dotenv').config()```
6. Whenever you need to sign or verify JWT, use the secret from the environment variables, keep key secure and easily configureable: ```const jwtSecret = process.env.JWT_SECRET;```

## Good Practices
- Never commit `.env`, always keep `.env` in `.gitignore`, to prevent it being pushed to version control
- Reguarly rotate secret keys for good security
