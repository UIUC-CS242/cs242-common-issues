# Common issues with Assignment 3.0

## React

#### Do not write API requests directly in components

#### Do not use global variables as an alternative to state

#### Do not use inline styles

#### Combine state hooks if they come from the same source

#### Define stateful functions inside the component

## General coding issues

#### Use `const` instead of `let` whenever possible

#### Use template literals over string concatenation

###### Bad

```ts
const repoCount = "Repo count: " + profile.repoCount;
```

```tsx
<Text>{"Repo count: " + profile.repoCount}</Text>
```

###### Good

```ts
const repoCount = `Repo count ${profile.repoCount}`;
```

```tsx
<Text>{`Repo count: ${profile.repoCount}`}</Text>
```

#### Use spreads

#### Use `async`/`await` instead of `Promise` chaining

#### Use a date package

## TypeScript

#### Do not use the `any` type

###### Bad

```ts
const profile: any = {
  username: "jtaylorchang",
  repositories: [
    {
      name: "sp21-cs242-assignment3",
    },
  ],
};
```

###### Good

```ts
interface Repository {
  name: string;
}

interface Profile {
  username: string;
  repositories: Repository[];
}

const profile: Profile = {
  username: "jtaylorchang",
  repositories: [
    {
      name: "sp21-cs242-assignment3",
    },
  ],
};
```

#### Use correct types and keywords

###### Bad

```ts
interface Profile {
  name: String;
  repoCount: Number;
  repos: Array<Repository>;
}
```

###### Good

```ts
interface Profile {
  name: string;
  repoCount: number;
  repos: Repository[];
}
```

#### Use the `as` keyword instead of casting

###### Bad

```ts
const profile = <Profile>response.json();
```

###### Good

```ts
const profile = response.json() as Profile;
```
