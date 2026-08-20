---
name: Password hashing
description: Password hashing uses bcrypt (saltRounds=10); seed script must be re-run or hash updated when changing algorithm
---

# Password Hashing

## Rule
`hashPassword` in `artifacts/api-server/src/routes/auth.ts` uses **bcrypt** with `saltRounds=10` and a per-password salt (built into bcrypt). Do NOT use SESSION_SECRET or any fixed salt — bcrypt handles salt internally.

**Why:** SHA-256 with a static salt is a fast hash vulnerable to offline cracking if the DB is leaked. Bcrypt is the standard for password storage.

## Important
- Admin seed hash in the DB must be regenerated whenever the hashing algorithm changes
- Use the seed file (`artifacts/api-server/src/db/seed.ts`) to compute and store the correct hash
- The `verifyPassword` function must use `bcrypt.compare()`, not a hash comparison
