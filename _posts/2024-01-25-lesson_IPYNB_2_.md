---
comments: True
layout: notebook
title: Deployment
description: An in depth deployment lesson
type: hacks
toc: True
courses: {'csa': {'week': 18}}
---

# Deployment Mini Guide

## Mini-Guide: Deploying Your Site with AWS

### Prerequisites
1. **GitHub Repository:**
   - Must have a backend repository on GitHub.

2. **Docker Setup:**
   - Verify Backend Docker files are running on localhost.

3. **Domain Configuration:**
   - Have a configured Domain Name pointing to the Public IP of your deployment server using AWS Route 53.

### AWS EC2 Access
1. **Login to AWS Console:**
   - Access AWS Management Console.
   - Navigate to "EC2" and select "Instances."

2. **Instance Selection:**
   - Choose the appropriate instance (CSP or CSA) based on your project.

3. **Terminal Access:**
   - Access the deployment server using either csp.nighthawkcodingsociety.com or csa.nighthawkcodingsociety.com.
   - Use the username "ubuntu" with the password hint "3 Musketeers."

### Application Setup
1. **Finding Port:**
   - Run `docker ps` on AWS EC2 terminal to find an available port starting with 8—.
   - Explanation: Identifies an available port for the application.

2. **Local Docker Setup:**
   - Configure Docker files in VSCode on localhost using the chosen port.
   - Explanation: Sets up the local environment for Docker.

3. **Testing:**
   - Run `docker-compose up` or `sudo docker-compose up` in VSCode terminal.
   - Open `http://localhost:8---` in your browser to check if it runs smoothly.
   - Explanation: Tests the application locally before deployment.

### Server Setup
1. **AWS EC2 Terminal:**
   - Clone your backend repo: `git clone github.com/server/project.git my_unique_name`.
   - Navigate to the repo: `cd my_unique_name`.
   - Explanation: Sets up the server environment and fetches the project code.

2. **Build and Test:**
   - Build the site: `docker-compose up -d --build`.
   - Test your site: `curl localhost:8---` (replace '8---' with your port).
   - Explanation: Builds and tests the application on the server.

### DNS & NGINX Setup
1. **Route 53 DNS:**
   - Set up DNS subdomain for your backend server in AWS Route 53.
   - Explanation: Configures DNS for domain mapping.

2. **NGINX Configuration:**
   - Navigate to `/etc/nginx/sites-available` in the terminal.
   - Create an NGINX config file and configure it accordingly.
   - Explanation: Configures NGINX as a reverse proxy for the application.

3. **Validation and Restart:**
   - Validate with `sudo nginx -t`.
   - Restart NGINX: `sudo systemctl restart nginx`.
   - Test your domain on your desktop browser (http://).
   - Explanation: Validates NGINX configuration and restarts for changes to take effect.

### Certbot Configuration
1. **Run Certbot:**
   - Execute `sudo certbot --nginx` and follow prompts.
   - Choose appropriate options for HTTPS activation.
   - Explanation: Configures SSL certificates for secure communication.

2. **Verify HTTPS:**
   - Test your domain in the browser using HTTPS.
   - Explanation: Ensures successful HTTPS setup.

### Changing Code and Deployment Updates
1. **VSCode Changes:**
   - Before updating, run `git pull` to sync changes.
   - Make code changes and test using Docker Desktop.
   - Explanation: Synchronizes code changes and tests locally.

2. **Deployment Update:**
   - If all goes well, sync changes via UI or `git push` from the terminal.
   - Explanation: Deploys updated code to the server.

### Pulling Changes into AWS EC2
1. **AWS EC2 Terminal:**
   - Navigate to your repo: `cd ~/my_unique_name`.
   - Stop the server: `docker-compose down`.
   - Pull changes: `git pull`.
   - Rebuild and start the container: `docker-compose up -d --build`.
   - Explanation: Pulls and updates code on the server, then restarts the application.

### Optional Troubleshooting Checks on AWS EC2
1. **Check Server Status:**
   - Use commands like `curl localhost:8---` and `docker-compose ps` for verification.
   - Explanation: Checks the status of the running application.

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
