# ☕ CoffeeQL

> The query language for everything structured, unstructured, geo, and AI in one syntax.

```coffeeql
users[]
  .where(age > 18, city = "Delhi")
  .give(name, email)
  .sort(balance, DESC)
  .cup(10)
```

## Install

```bash
npm i coffeeql
```

## Usage

```javascript
const { coffeeql, isValid } = require('coffeeql')

const result = coffeeql('users[].where(age > 18).give(name).cup(10)')
console.log(result.success) // true
```

## Why CoffeeQL?

- `users[]` → structured (SQL)
- `products{}` → unstructured (MongoDB)
- `.near()` → geo built-in
- `.like()` → AI similarity built-in
- One syntax for everything

## Links

- 📖 **Docs** → [coffeeql.dev](https://coffeeql.dev)
- 📦 **npm** → [npmjs.com/package/coffeeql](https://npmjs.com/package/coffeeql)
- ⭐ **GitHub** → [github.com/KhushviB/coffeeql](https://github.com/KhushviB/coffeeql)

## License

MIT © CoffeeQL
