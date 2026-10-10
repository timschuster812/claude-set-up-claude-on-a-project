# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Description

A REST API for user management built with Express.js

## Commands

- `npm install` — install dependencies (CI uses Node 22)
- `npm run dev` — start the API with `node --watch` on http://localhost:3000
- `npm start` — start the API without watching
- `npm test` — run all tests (`node --test`)
- `npm test -- tests/users.test.js` — run a single test file
- `npm test -- --test-name-pattern="GET /users/:id"` — run tests whose name matches a pattern
- `npm run lint` — ESLint over the whole repo

CI (`.github/workflows/ci.yml`) runs lint then test on every push and pull request.

## Architecture

- `server.js` builds the Express `app`, mounts routers at `/users` and `/health`, and exports `app`. It calls `listen` only when run directly (`require.main === module`), so tests import `app` without opening a port. Keep that guard when changing the entry point.
- `routes/<resource>.js` each export an `express.Router()`. Handlers do request validation and status codes, then call `db/store.js` directly. There is no service layer.
- `db/store.js` is the only data layer: a module-level in-memory array with `getAllUsers`, `getUserById`, and `createUser`. Its state lives for the life of the process and is shared by every request and test, so tests that write data affect later tests in the same run. Swapping in a real database means changing this module's interface, not the routes.
- IDs are numeric. Routes convert `req.params.id` with `Number(...)` before lookup, so a non-numeric id simply returns 404.
- `PORT` is read from `process.env`. Nothing loads `.env` automatically; there is no dotenv dependency, so `.env.example` documents config but the app only sees variables set in the shell.

## Conventions

- Use `require` / `module.exports` (CommonJS), not ES `import` / `export` modules.
- Use the parameter names `_`, `req`, `res`, or `next` for unused Express handler arguments, not other unused names, so ESLint stays quiet.
- Use `node:test` with `assert` and `supertest` against the exported `app`, not a running server.
- Use `Number(req.params.id)` for route IDs, not string IDs, because stored IDs are numeric.
