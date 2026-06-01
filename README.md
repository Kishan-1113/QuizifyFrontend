# Quizify - Interactive Quiz Platform (Frontend UI)

Welcome to the **Quizify Frontend UI**! This repository contains the modern, responsive, and feature-rich Single Page Application (SPA) for the Quizify platform. Previously hosted within a Spring Boot application (`resources/static`), it has now been successfully separated into a clean static client codebase optimized for standalone local development and production cloud deployment (e.g., **Vercel**).

---

## 🌟 Key Features

Quizify delivers a premium, dynamic web experience using modern typography, glassmorphism card designs, smooth animations, and tailored color schemes:

- **Sleek Theme System**: Toggle seamlessly between dark mode (default slate/neon accent theme) and light mode. Your preference is persisted in local storage.
- **Fluid SPA Transitions**: Smooth layout transitions and screens switching without page reloads.
- **Dynamic Quiz Generator**: Customize quizzes based on title, category, difficulty level, and number of questions.
  - *Direct Play (Transient)*: Generates a temporary quiz session stored in memory.
  - *Create & Save (Persistent)*: Saves the quiz layout to the database, copying the unique Quiz ID to your clipboard for sharing.
- **Public & Private Sharing**: Play shared quizzes by typing in a Quiz ID. Access control protects private quizzes from unauthorized play.
- **Real-Time Visual Timer**: A beautiful radial SVG animation tracks seconds remaining for each question (e.g., 30s) and automatically advances/submits on timeouts.
- **Detailed Quiz Reviews**: Shows fraction/percentage circular score charts, performance headline badges, and a checklist detailing your answer vs. the correct answer.
- **Interactive Stats & Ratings**: Log in to view your quiz history logs, scores, correct/incorrect statistics, and completion dates.
- **Full Admin Dashboard**:
  - *Questions Tab*: Complete CRUD operations (Add, Edit, View details, Delete) for the question bank. Includes autocomplete datalists and search filters.
  - *Quizzes Tab*: Overview of all persistent quizzes stored in the database, including title, categories, creator emails, visibility scopes, and deletion.

---

## ⚙️ API Configuration Settings

To ensure the separated UI client communicates correctly with your backend server (local or deployed), configuration is defined directly in the source code of **[app.js](file:///c:/Users/Kishan%20Bhadra/Desktop/JavaWebProject/New%20folder/app.js)**:

1. **Automatic Development Default**: When running the UI on `localhost` or `127.0.0.1`, it automatically defaults `API_BASE` to your local Spring Boot port (`http://localhost:8080`).
2. **Production Backend URL Configuration**: When deployed in production (e.g. Vercel), it defaults to the URL defined by the `BACKEND_PROD_URL` constant.
3. **How to configure**: Open `app.js` and edit the `BACKEND_PROD_URL` constant at the top of the file to point to your deployed Spring Boot server:
   ```javascript
   const BACKEND_PROD_URL = "https://your-deployed-backend-api.com";
   ```

---

## 💻 Local Development Setup

To run the frontend UI locally on your computer, we recommend running a local web server (instead of opening `index.html` directly from files, which can cause CORS or browser security issues with network requests):

### Option 1: Live Server (VS Code)
If you use Visual Studio Code, install the popular **Live Server** extension, right-click `index.html`, and select **Open with Live Server**. It will host the frontend at `http://127.0.0.1:5500`.

### Option 2: Node.js (npx)
If you have Node.js installed, open your terminal in this directory and run:
```bash
npx serve
```
This will host your application instantly, usually at `http://localhost:3000` or `http://localhost:5000`.

### Option 3: Python HTTP Server
If you have Python installed, run one of the following commands in your terminal:
- **Python 3.x**:
  ```bash
  python -m http.server 3000
  ```
- **Python 2.x**:
  ```bash
  python -m SimpleHTTPServer 3000
  ```
Open `http://localhost:3000` in your web browser.

---

## 🚀 Deployment on Vercel

Vercel is the ideal hosting provider for static HTML/JS/CSS applications. You can deploy this UI in minutes for free.

### Method 1: Deploy via Vercel Dashboard (Recommended)

1. Push your frontend code to a Git repository (GitHub, GitLab, or Bitbucket).
2. Log in to [Vercel](https://vercel.com/) and click **Add New** > **Project**.
3. Import your Git repository.
4. Under **Project Settings**:
   - **Framework Preset**: Choose **Other** (since this is vanilla JS/HTML).
   - **Build and Output Settings**: Leave all settings at their defaults (Build command: empty, Output directory: default root `.`).
5. Click **Deploy**. Vercel will build and assign you a public production URL (e.g. `quizify-client.vercel.app`).

### Method 2: Deploy via Vercel CLI

If you prefer deploying from your command line:
1. Install the Vercel CLI globally:
   ```bash
   npm install -g vercel
   ```
2. Navigate to this project directory and run:
   ```bash
   vercel
   ```
3. Follow the prompts to log in, link your project, and deploy.
4. To promote to production, run:
   ```bash
   vercel --prod
   ```

---

## 🛡️ Crucial: Spring Boot Backend CORS Configuration

Because your frontend (e.g., hosted on Vercel at `https://quizify.vercel.app`) is now on a different domain than your backend (e.g., hosted at `https://quizify-backend.herokuapp.com` or `http://localhost:8080`), browser security will block all API calls unless you enable **CORS (Cross-Origin Resource Sharing)** on your Spring Boot application.

Add one of the following configurations to your Spring Boot project:

### Approach A: Global Configuration (Recommended)
Add a configuration class inside your Spring Boot codebase (e.g., under a `config` package) to allow requests from your Vercel domains:

```java
package com.quizify.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**") // Enable CORS for all API paths
                        .allowedOrigins(
                            "http://localhost:3000",      // Node local server
                            "http://localhost:5000",      // serve local port
                            "http://127.0.0.1:5500",      // VS Code Live Server
                            "https://your-vercel-app.vercel.app" // Your deployed Vercel domain
                        )
                        .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                        .allowedHeaders("*")
                        .allowCredentials(true);
            }
        };
    }
}
```

### Approach B: Annotation Level (`@CrossOrigin`)
If you want to configure CORS on specific Controllers, add the `@CrossOrigin` annotation to your controllers:

```java
@RestController
@RequestMapping("/quiz")
@CrossOrigin(origins = {"http://localhost:3000", "https://your-vercel-app.vercel.app"}, allowCredentials = "true")
public class QuizController {
    // Endpoints
}
```

### Approach C: Spring Security CORS integration
If you are using **Spring Security** (for JWT authentication, which Quizify uses), make sure to configure CORS within your security filter chain config:

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        .cors(cors -> cors.configurationSource(request -> {
            var corsConfiguration = new org.springframework.web.cors.CorsConfiguration();
            corsConfiguration.setAllowedOrigins(List.of("http://localhost:3000", "https://your-vercel-app.vercel.app"));
            corsConfiguration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
            corsConfiguration.setAllowedHeaders(List.of("*"));
            corsConfiguration.setAllowCredentials(true);
            return corsConfiguration;
        }))
        // ... rest of security filter chain configurations
        .csrf(csrf -> csrf.disable());
    return http.build();
}
```
