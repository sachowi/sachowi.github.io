---
layout: post
title: How To Make Requests from A React.js Application to A Java Backend Using Sessions
---

To make requests from a React.js application to a Java backend using sessions, you'll need to handle session management and communication between the client (React) and the server (Java). Below is a step-by-step guide on how to implement this setup effectively.

### 1. **Setting Up the Java Backend**

#### **1.1 Configure Session Management**

In a Java backend, typically using Spring Boot, you would configure session management to store session data. Here’s a basic setup:

```java
// Add Spring Boot dependencies to your build.gradle or pom.xml
// Example for Gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-session'
}
```

**Create a configuration class to enable HTTP session management:**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.session.web.http.HttpSessionIdResolver;
import org.springframework.session.web.http.HttpSessionIdResolver.CookieHttpSessionIdResolver;

@Configuration
public class SessionConfig {

    @Bean
    public HttpSessionIdResolver httpSessionIdResolver() {
        return new CookieHttpSessionIdResolver(); // Use cookies for session management
    }
}
```

**Create a controller to handle requests and manage sessions:**

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpSession;

@RestController
public class UserController {

    @PostMapping("/login")
    public String login(@RequestParam String username, HttpSession session) {
        // Store user information in the session
        session.setAttribute("user", username);
        return "Logged in";
    }

    @GetMapping("/profile")
    public String profile(HttpSession session) {
        // Retrieve user information from the session
        String user = (String) session.getAttribute("user");
        return user != null ? "Welcome " + user : "Not logged in";
    }
}
```

### 2. **Setting Up the React Frontend**

#### **2.1 Install Axios for Making HTTP Requests**

Install Axios (or use Fetch) for making HTTP requests from React:

```bash
npm install axios
```

#### **2.2 Configure Axios to Send Cookies**

To handle sessions, you need to make sure that cookies are sent with each request. Configure Axios accordingly:

```javascript
// src/api/axios.js
import axios from 'axios';

const instance = axios.create({
    baseURL: 'http://localhost:8080', // Base URL of your Java backend
    withCredentials: true, // Enable sending cookies with requests
});

export default instance;
```

#### **2.3 Making Requests and Handling Sessions**

Here’s how you can perform login and fetch user profile data:

```javascript
// src/App.js
import React, { useState } from 'react';
import axios from './api/axios';

function App() {
    const [username, setUsername] = useState('');
    const [message, setMessage] = useState('');

    const handleLogin = async () => {
        try {
            const response = await axios.post('/login', `username=${username}`, {
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded'
                }
            });
            setMessage(response.data);
        } catch (error) {
            setMessage('Error logging in');
        }
    };

    const fetchProfile = async () => {
        try {
            const response = await axios.get('/profile');
            setMessage(response.data);
        } catch (error) {
            setMessage('Error fetching profile');
        }
    };

    return (
        <div>
            <h1>React and Java Session Example</h1>
            <input
                type="text"
                value={username}
                onChange={(e) => setUsername(e.target.value)}
                placeholder="Enter username"
            />
            <button onClick={handleLogin}>Login</button>
            <button onClick={fetchProfile}>Fetch Profile</button>
            <p>{message}</p>
        </div>
    );
}

export default App;
```

### 3. **Cross-Origin Resource Sharing (CORS)**

If your React app and Java backend are running on different domains or ports, you need to handle CORS to allow cross-origin requests.

**Configure CORS in the Java backend:**

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:3000") // Your React app's URL
                .allowedMethods("GET", "POST")
                .allowCredentials(true);
    }
}
```

### 4. **Testing and Debugging**

1. **Ensure Sessions Are Managed**: Use browser developer tools to check if cookies are being sent with requests and if sessions are being managed correctly.

2. **Check for CORS Issues**: Ensure that CORS is properly configured if you encounter any cross-origin request issues.

3. **Handle Errors Gracefully**: Implement error handling in both your Java backend and React frontend to provide clear feedback to users.

By following these steps, you can effectively manage sessions between a React.js frontend and a Java backend, ensuring a smooth and secure user experience.
