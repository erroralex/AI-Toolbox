# Contributing to Latent Library

We welcome contributions! Please follow these guidelines.

## Development Setup

1.  **Prerequisites:**
    *   Java 21+ (JDK)
    *   Node.js 20+ & npm
    *   No local Maven install needed — use the bundled wrapper (`./mvnw`).

2.  **Backend:**
    *   Navigate to `backend/`.
    *   Run `./mvnw clean install`.
    *   Run `./mvnw spring-boot:run`.
    *   Run tests with `./mvnw test`.

3.  **Frontend:**
    *   Navigate to `frontend/`.
    *   Run `npm install`.
    *   Run `npm run dev`.

4.  **Electron:**
    *   Ensure the backend jar is built (`cd backend && ./mvnw clean package -DskipTests`).
    *   Navigate to `electron/`.
    *   Run `npm install`.
    *   Run `npm start`.
    *   Note: this launches its own backend process from `backend/target/backend.jar`. If you already have the backend
        running separately (e.g. via an IDE), you'll end up with two backend processes — that's expected, not a bug.
    *   If you change frontend code, rebuild both the frontend (`npm run build`) *and* the backend jar
        (`./mvnw clean package -DskipTests`) before relaunching Electron standalone — the jar bundles a snapshot of the
        frontend build, so a frontend-only rebuild won't be reflected until the jar is repackaged.

See [AGENTS.md](AGENTS.md) for the full engineering rulebook (testing contracts, git conventions, module boundaries, etc.)
that applies to all contributions, human or AI-assisted.

## Code Style

*   **Java:** Follow standard Java conventions. Use 4 spaces for indentation.
*   **Vue/JS:** Use 2 spaces for indentation. Follow Vue 3 Composition API best practices.

## Pull Requests

*   Create a feature branch.
*   Submit a PR with a clear description of changes.
