# Common issues with Assignment 3.0

## Repository Structure

You should be logically separating your code into directories.

###### Bad

```
App.tsx
Repository.tsx
List.tsx
api-code.ts
test.App.js
package.json
package-lock.json
.eslintrc.json
```

###### Good

```
src/
  App.tsx
  screens/
    Profile.tsx
    Repository.tsx
    Followers.tsx
    Following.tsx
  components/
    User/
      UserBio.tsx
      UserList.tsx
      UserListItem.tsx
    Repository/
      RepositoryList.tsx
      RepositoryListItem.tsx
  github-api/
    query-profile.ts

__mocks__/
  src/
    github-api/
      query-profile.ts

tests/
  test.api.ts
  test.repository.ts
  test.profile.ts
  test.navigation.ts

.eslintrc.json
tsconfig.json
jest.config.ts
package.json
package-lock.json
```

## Eslint

#### Disabling eslint rules

Some students disable eslint rules that we required. If they do this, they cannot get credit for eslint on the rubric.

###### Bad

```ts
/* eslint-disable some rule here */
```

###### Good

Code that follows our eslint configuration.

## React

#### Components should have doc comments

Components should have standard doc comments that explain what the component is as well as what the props are

###### Bad

```tsx
const Repository: React.FC<{ repo: Repo }> = ({ repo }) => {
  return (
    <View>
      <Text>{repo.name}</Text>
    </View>
  );
};
```

###### Good

```tsx
/**
 * A component that renders a repository.
 *
 * @param repo The repository from the GitHub API
 */
const Repository: React.FC<{ repo: Repo }> = ({ repo }) => {
  return (
    <View>
      <Text>{repo.name}</Text>
    </View>
  );
};
```

#### Use functional components instead of class components

Students should be using functional components whenever possible.

###### Bad

```tsx
class ProfileScreen extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      profile: null,
    };
  }

  componentDidMount() {
    (async () => {
      const profile = await getProfile();

      this.setState(() => ({
        profile,
      }));
    })();
  }

  render() {
    return (
      <View>
        <Text>{profile?.username ?? "Loading..."}</Text>
      </View>
    );
  }
}
```

###### Good

```tsx
const ProfileScreen: React.FC = () => {
  const [profile, setProfile] = React.useState<Profile | undefined>();

  React.useEffect(() => {
    (async () => {
      const profileData = await getProfile();

      setProfile(() => profileData);
    })();
  }, []);

  return (
    <View>
      <Text>{profile?.username ?? "Loading..."}</Text>
    </View>
  );
};
```

#### Missing dependencies for hooks

Hooks must have all dependencies used in the hook listed in the array, no more, no less. If there are no dependencies, there should be an empty array. Supplying no array means everything becomes a dependency (which is rarely the desired result).

###### Bad

```tsx
useEffect(() => {
  dispatch(action.getProfile());
}, []);
```

###### Good

```tsx
useEffect(() => {
  dispatch(action.getProfile());
}, [dispatch]);
```

#### Manually assembling DOM

Students should inline assemble DOM elements whenever possible rather than building the DOM in multiple pieces.

###### Bad

```tsx
const ProfileScreen: React.FC = () => {
  const [profile, setProfile] = React.useState<Profile | undefined>();

  React.useEffect(() => {
    (async () => {
      const profileData = await getProfile();

      setProfile(() => profileData);
    })();
  }, []);

  const repositories = [];

  profile?.repositories.forEach((repo) => {
    repositories.push(<Repository repo={repo} />);
  });

  return (
    <View>
      <Text>{profile?.username ?? "Loading..."}</Text>

      {repositories}
    </View>
  );
};
```

###### Good

```tsx
const ProfileScreen: React.FC = () => {
  const [profile, setProfile] = React.useState<Profile | undefined>();

  React.useEffect(() => {
    (async () => {
      const profileData = await getProfile();

      setProfile(() => profileData);
    })();
  }, []);

  return (
    <View>
      <Text>{profile?.username ?? "Loading..."}</Text>

      {profile?.repositories.map((repo) => (
        <Repository key={repo.id} repo={repo} />
      ))}
    </View>
  );
};
```

#### Spread props inside the component declaration

Students should spread their props inside of the functional component declaration. This makes the definition more clear, allows for default values, and avoids constantly accessing the props variable. This is especially crucial if they are not using TypeScript.

###### Bad

```tsx
const ProfileScreen = (props) => {
  const openPage = (page: string) => {
    props.navigation.navigate(page);
  };

  return (
    <View>
      <Text onPress={() => openPage("Repositories")}>Repositories</Text>
    </View>
  );
};
```

###### Good

```tsx
const ProfileScreen = ({ navigation }) => {
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

#### Do not write API requests directly in components

Students should never write their API requests directly inside their components. The API request shoud be extracted into a helper function in a separate file and imported into the component file. If you see a student calling `fetch` directly in the component, chances are they need to improve their decomposition.

###### Bad

```tsx
const ProfileScreen: React.FC = () => {
  const [profile, setProfile] = React.useState<Profile | undefined>();

  React.useEffect(() => {
    (async () => {
      const res = await fetch("http....");
      const profileData = await res.json();

      setProfile(profileData);
    })();
  }, []);

  return (
    <View>
      <Text>{profile?.username ?? "Loading..."}</Text>
    </View>
  );
};
```

###### Good

```tsx
const ProfileScreen: React.FC = () => {
  const [profile, setProfile] = React.useState<Profile | undefined>();

  React.useEffect(() => {
    (async () => {
      const profileData = await getProfile();

      setProfile(profileData);
    })();
  }, []);

  return (
    <View>
      <Text>{profile?.username ?? "Loading..."}</Text>
    </View>
  );
};
```

###### Good

#### Do not use global variables as an alternative to state

Students should not store stateful data in globals that are imported into components. This is a very severe violation of the fundamental principles of React and should be worth decomposition points proportional to the violation's severity. The correct answer is to use global state management via a library like `Redux`. If `Redux` is too difficult for them, they can use a `Context` or at a minimum store their data inside state and pass it as props. `Redux` is greatly preferred.

###### Bad

```tsx
let profile: Profile | undefined;

const ProfileScreen: React.FC = () => {
  React.useEffect(() => {
    (async () => {
      profile = await getProfile();
    })();
  }, []);

  return (
    <View>
      <Text>{profile?.username ?? "Loading..."}</Text>
    </View>
  );
};
```

###### Good

```tsx
import { useSelector, useDispatch } from "react-redux";
import { ReduxState, actions } from "./redux";

const ProfileScreen: React.FC = () => {
  const profile = useSelector((state: ReduxState) => state.github.profile);

  const dispatch = useDispatch();
  const dispatchGetProfile = React.useCallback(
    () => dispatch(actions.getProfile()),
    [dispatch]
  );

  React.useEffect(() => {
    dispatchGetProfile();
  }, []);

  return (
    <View>
      <Text>{profile?.username ?? "Loading..."}</Text>
    </View>
  );
};
```

#### Do not use inline styles

Students should not hardcode styles inline. Instead they should define styles in a `StyleSheet`. This is a reusability issue as well as a refactoring one and often indicates poor decomposition / design.

###### Bad

```tsx
<Text style={{ fontWeight: 600 }}>Repositories:</Text>
```

###### Good

```tsx
const styles = StyleSheet.create({
  boldText: {
    fontWeight: 600,
  },
});
```

```tsx
<Text style={styles.boldText}>Repositories</Text>
```

#### Combine state hooks if they come from the same source

Using a lot of separate state hooks for data that comes from the same source (ie is always set at the same time) is unnecessarily complex and makes code harder to follow as well as clutters the namespace. It also means they have to call a ton of separate setters to update each piece individually. Instead students should use a single `useState` hook for the entire object, often the user's profile. If they are worried about triggering re-renders, they can look into using the `useMemo` hook.

###### Bad

```tsx
const [username, setUsername] = React.useState<string>("");
const [name, setName] = React.useState<string>("");
const [imageUrl, setImageUrl] = React.useState<string>("");

React.useEffect(() => {
  (async () => {
    const profile = await getProfile();

    setUsername(profile.username);
    setName(profile.name);
    setImageUrl(profile.imageUrl);
  })();
}, []);
```

###### Good

```tsx
const [profile, setProfile] = React.useState<Profile | undefined>();

React.useEffect(() => {
  (async () => {
    const profile = await getProfile();

    setProfile(profile);
  })();
}, []);
```

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

#### Writing code that only works in web

Students seem to be writing code that uses web-only packages such as `css` or regular `html` elements like `<img />`. This is **invalid** React-Native code. They must only use React-Native specific components or their app will work in the web browser but will fail on a device.

###### Bad

```tsx
<img src={imageUrl} style={styles.image} />
```

###### Good

```tsx
import { Image } from "react-native";

<Image source={{ uri: imageUrl }} style={styles.image} />;
```

## General coding issues

#### Use `async`/`await` instead of `Promise` chaining

Students should use `async`/`await` over `Promise` chaining. It produces easier to follow code as well as avoids creating unnecessary nested scopes.

###### Bad

```ts
const getProfile = (): Promise<Profile | string> => {
  return new Promise<Profile | string>((resolve) => {
    let errorMessage = "";
    let responseJson;

    fetch()
      .then((res) => res.json())
      .then((res) => {
        responseJson = res;
      })
      .catch((error) => {
        errorMessage = error.message;
      })
      .finally(() => {
        if (responseJson) {
          resolve(responseJson);
        } else {
          resolve(errorMessage);
        }
      });
  });
};
```

###### Good

```ts
const getProfile = async (): Profile | string => {
  try {
    const response = await fetch();
    const responseJson = await response.json();

    return responseJson;
  } catch (error) {
    return error.message ?? "";
  }
};
```

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

###### Bad

```ts
const getProfileWithCount = async () => {
  let data = await getProfile();
  data.repoCount = data.repositories.length;

  return data;
};
```

###### Good

```ts
const getProfileWithCount = async () => {
  const data = await getProfile();

  return {
    ...data,
    repoCount: data.repositories.length,
  };
};
```

#### Use a date package

Students should not manually manipulate `Date` objects or timestamps. If they need to format or process a date, they should use a dedicated library like `moment` or `date-fns`. Both libraries have similar functionality but `date-fns` focuses on using the built in `Date` object and functions that accept the date and return the result rather than mutating a class. This is better than using `moment` both for this immutability but also because `moment` is not compatible with `tree-shaking`.

###### Bad

```ts
const creationDate = profile.creationDate.toISOString().split("T")[0];
```

###### Good

```ts
import moment from "moment";

const creationDate = moment(profile.creationDate).format("YYYY-MM-DD");
```

#### Watch out for strange spacing

Students should be following the `airbnb` style guide. For good opinionated styling, students can use `Prettier`.

###### Bad

```ts
interface Profile {
  // prettier-ignore
  username : string;
  // prettier-ignore
  name:string;
}

// prettier-ignore
const username = profile? profile.username : "N/A";
```

```ts
interface Profile {
  username: string;
  name: string;
}

const username = profile ? profile.username : "N/A";
```

#### Use optional chaining

Students should take advantage of optional chaining instead of using explicit if statements to verify a chain.

###### Bad

```ts
let repositories: Repository[] = [];

const data = getProfile();

if (data && data.user && data.user.repositories) {
  repositories = data.user.repositories.nodes;
}
```

###### Good

```ts
const data = getProfile();

const repositories: Repository[] = data?.user?.repositories?.nodes;
```

#### Use the nullish coalescing operator

The nullish coalescing operator should be used as a shorthand for the equivalent `!== undefined` or `!== null` check when a default value is desired.

###### Bad

```ts
const username = profile?.username ? profile?.username : "N/A";
```

###### Good

```ts
const username = profile?.username ?? "N/A";
```

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
const profile = <Profile>await response.json();
```

###### Good

```ts
const profile = (await response.json()) as Profile;
```
