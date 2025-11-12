# Feature Implementation Plan: AI Quote Generation

## 📋 Todo Checklist
- [x] Add Spring AI dependencies to `pom.xml`.
- [x] Configure Vertex AI Gemini in `application.yml`.
- [x] Create the `AiQuoteService`.
- [x] Add the new endpoint to `QuoteController`.
- [x] Final Review and Manual Testing.

## 🔍 Analysis & Investigation

### Codebase Structure
The codebase is a standard Spring Boot application.
- `pom.xml`: Manages dependencies.
- `src/main/resources/application.yml`: Main configuration file.
- `src/main/java/com/example/quotes/domain`: Contains the domain objects and services.
- `src/main/java/com/example/quotes/web`: Contains the REST controllers.

### Current Architecture
The application follows a classic 3-tier architecture:
1.  **Controller Layer** (`QuoteController`): Handles HTTP requests.
2.  **Service Layer** (`QuoteService`): Contains business logic.
3.  **Repository Layer** (`QuoteRepository`): Interacts with the database.

The new `AiQuoteService` will be part of the service layer and will be called by the `QuoteController`.

### Dependencies & Integration Points
- **Spring AI**: The core dependency for this feature. We will use `spring-ai-starter-vertex-ai-gemini`.
- **Vertex AI**: The Google Cloud service that provides the Gemini model. The application will connect to Vertex AI using the credentials configured in the environment.
- `QuoteController`: The new endpoint will be added to this controller.
- `Quote` domain object: The `AiQuoteService` will generate a `Quote` object.

### Considerations & Challenges
- **Authentication**: The application needs to be authenticated with Google Cloud to use Vertex AI. This is expected to be handled via Application Default Credentials (ADC) in the execution environment.
- **Prompt Engineering**: The quality of the generated quotes will depend on the prompt. The initial prompt is "Generate one single short, witty random quote based on the theme in {theme}". This might need to be refined based on the results.
- **Error Handling**: The plan should include error handling for cases where the AI model fails to generate a quote.

## 📝 Implementation Plan

### Prerequisites
- Google Cloud project with Vertex AI API enabled.
- `gcloud` CLI authenticated with the project.

### Step-by-Step Implementation
1.  **Step 1: Update `pom.xml`**
    -   **Files to modify**: `pom.xml`
    -   **Changes needed**: Add the `spring-ai-bom` to the `dependencyManagement` section and the `spring-ai-starter-vertex-ai-gemini` dependency to the `dependencies` section.

    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.ai</groupId>
                <artifactId>spring-ai-bom</artifactId>
                <version>1.1.0-M3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        ...
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-vertex-ai-gemini</artifactId>
        </dependency>
        ...
    </dependencies>
    ```

2.  **Step 2: Configure Vertex AI in `application.yml`**
    -   **Files to modify**: `src/main/resources/application.yml`
    -   **Changes needed**: Add the configuration for the Spring AI Vertex AI Gemini model.

    ```yaml
    spring:
      ai:
        vertex:
          ai:
            gemini:
              project-id: ${GOOGLE_CLOUD_PROJECT}
              location: ${GOOGLE_CLOUD_LOCATION}
              chat:
                options:
                  model: gemini-2.5-flash-lite
                  max-output-tokens: 64
    ```

3.  **Step 3: Create `AiQuoteService.java`**
    -   **Files to create**: `src/main/java/com/example/quotes/domain/AiQuoteService.java`
    -   **Changes needed**: Create a new service that uses `ChatClient` to generate a quote from a given theme.

    ```java
    package com.example.quotes.domain;

    import org.springframework.ai.chat.client.ChatClient;
    import org.springframework.stereotype.Service;

    @Service
    public class AiQuoteService {

        private final ChatClient chatClient;

        public AiQuoteService(ChatClient.Builder chatClientBuilder) {
            this.chatClient = chatClientBuilder.build();
        }

        public Quote generateQuote(String theme) {
            String prompt = "Generate one single short, witty random quote based on the theme in " + theme;
            String quoteText = chatClient.prompt()
                    .user(prompt)
                    .call()
                    .content();
            return new Quote(quoteText, "AI");
        }
    }
    ```

4.  **Step 4: Update `QuoteController.java`**
    -   **Files to modify**: `src/main/java/com/example/quotes/web/QuoteController.java`
    -   **Changes needed**: Inject the `AiQuoteService` and add a new endpoint `/generate-quote`.

    ```java
    // Add this import
    import com.example.quotes.domain.AiQuoteService;

    @RestController
    public class QuoteController {

        private final QuoteService quoteService;
        private final AiQuoteService aiQuoteService; // Add this

        // Modify the constructor
        public QuoteController(QuoteService quoteService, AiQuoteService aiQuoteService) {
            this.quoteService = quoteService;
            this.aiQuoteService = aiQuoteService;
        }

        // Add this new method
        @GetMapping("/generate-quote")
        public Quote generateQuote(@RequestParam String theme) {
            return aiQuoteService.generateQuote(theme);
        }

        // ... existing methods
    }
    ```

### Testing Strategy
Since automated tests are out of scope for this plan, the feature will be tested manually.
1.  Run the application.
2.  Send a GET request to `http://localhost:8083/generate-quote?theme=programming`.
3.  Verify that the response is a JSON object representing a `Quote` with a witty quote about programming.

## 🎯 Success Criteria
- The application compiles and starts successfully.
- The `/generate-quote` endpoint is available.
- The endpoint returns a valid `Quote` object with a quote generated by the AI model based on the provided theme.
