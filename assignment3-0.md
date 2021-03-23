# Common issues with Assignment 3.0

## React

#### Use functional components instead of class components

Students should be using functional components whenever possible.

#### Spread props inside the component declaration

Students should spread their props inside of the functional component declaration. This makes the definition more clear, allows for default values, and avoids constantly accessing the props variable. This is especially crucial if they are not using TypeScript.

#### Do not write API requests directly in components

Students should never write their API requests directly inside their components. The API request shoud be extracted into a helper function in a separate file and imported into the component file. If you see a student calling `fetch` directly in the component, chances are they need to improve their decomposition.

#### Do not use global variables as an alternative to state

Students should not store stateful data in globals that are imported into components. This is a very severe violation of the fundamental principles of React and should be worth decomposition points proportional to the violation's severity. The correct answer is to use global state management via a library like `Redux`. If `Redux` is too difficult for them, they can use a `Context` or at a minimum store their data inside state and pass it as props. `Redux` is greatly preferred.

#### Do not use inline styles

Students should not hardcode styles inline. Instead they should define styles in a `StyleSheet`. This is a reusability issue as well as a refactoring one and often indicates poor decomposition / design.

#### Combine state hooks if they come from the same source

Using a lot of separate state hooks for data that comes from the same source (ie is always set at the same time) is unnecessarily complex and makes code harder to follow as well as clutters the namespace. It also means they have to call a ton of separate setters to update each piece individually. Instead students should use a single `useState` hook for the entire object, often the user's profile. If they are worried about triggering re-renders, they can look into using the `useMemo` hook.

#### Define stateful functions inside the component

Students should not write their functions outside of a component if they only depend on state or props. They should instead take advantage of scoping their functions to automatically have access.

###### Bad

```tsx
const ProfileScreen: React.FC<{ navigation: Navigation }> = ({
  navigation,
}) => {
  return (
    <View>
      <Text onPress={() => openPage(navigation, "Repositories")}>
        Repositories
      </Text>
    </View>
  );
};

const openPage = (navigation: Navigation, page: string) => {
  navigation.navigate(page);
};
```

###### Good

```tsx
const ProfileScreen: React.FC<{ navigation: Navigation }> = ({
  navigation,
}) => {
  const openPage = (page: string) => {
    navigation.navigate(page);
  };

  return (
    <View>
      <Text onPress={() => openPage("Repositories")}>Repositories</Text>
    </View>
  );
};
```

## General coding issues

#### Use `const` instead of `let` whenever possible

If a variable is not reassigned, it should always use `const` instead of `let`

###### Bad

```ts
let profile: Profile | null = null;

let res = await getProfile();

profile = res.profile;
```

###### Good

```ts
let profile: Profile | null = null;

const res = await getProfile();

profile = res.profile;
```

#### Use template literals over string concatenation

Students should use template literals instead of string concatenation. It is a more readable syntax and avoids mutations.

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

Students should be using object and array spreads whenever possible rather than doing direct mutations.

#### Use `async`/`await` instead of `Promise` chaining

Students should use `async`/`await` over `Promise` chaining. It produces easier to follow code as well as avoids creating unnecessary nested scopes.

#### Use a date package

Students should not manually manipulate `Date` objects or timestamps. If they need to format or process a date, they should use a dedicated library like `moment` or `date-fns`. Both libraries have similar functionality but `date-fns` focuses on using the built in `Date` object and functions that accept the date and return the result rather than mutating a class. This is better than using `moment` both for this immutability but also because `moment` is not compatible with `tree-shaking`.

## TypeScript

#### Do not use the `any` type

If a student is using TypeScript, they cannot use the `any` type or the `Object` type or `{}` or any equivalent escape type. This defeats the purpose of TypeScript because they are disabling types. I subtract one bonus point if a student uses any of those.

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

When using TypeScript, students should make sure that they use the correctly cased types as opposed to the classes for those types. If they use arrays, they should use the bracket notation instead of `Array<>`.

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

Sometimes a student will need to cast a value into a specific type, a common situation for this is when getting responses back from API requests. They should use the `as` keyword instead of casting.

###### Bad

```ts
const profile = <Profile>response.json();
```

###### Good

```ts
const profile = response.json() as Profile;
```

#### Watch out for strange spacing

Students should be following the `airbnb` style guide. For good opinionated styling, students can use `Prettier`.
