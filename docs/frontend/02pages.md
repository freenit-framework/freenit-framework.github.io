# Pages

Freenit comes with some of the most common pages implemented in the core library.

## Login

The minimal code to add login page to your site

```ts
<script lang="ts">
  import { Login } from 'freenit'
  import store from '$lib/store'
</script>

<Login store={store} />
```

## Register

The minimal code to add user registration page to your site

```ts
<script lang="ts">
  import { Register } from 'freenit'
  import store from '$lib/store'
</script>

<Register store={store} />
```

## Role

The page to add to the dashboard to have `Role` management, like adding and
removing users from the role. It is assumed that page is in `[pk]/+page.svelte`
file, hence the parameter `pk` in the code.

```ts
<script lang="ts">
  import { Role } from 'freenit'
  import store from '$lib/store'

  const { data } = $props()
</script>

<Role pk={data.pk} store={store} />
```

In order to get `pk` through props, we need to define `load` function.

```ts
export const load = ({ params }) => {
  return {
    pk: params.pk
  }
}
```

This function is defined in `+page.ts` file in the same directory where `+page.svelte` is. The same
function is used for `User` and `Theme` pages, so it will be omitted.

## Roles

The dashboard page for management of `Roles`.

```ts
<script lang="ts">
  import { Roles } from 'freenit'
  import store from '$lib/store'
</script>

<Roles store={store} />
```

## Theme

The page to add to the dashboard to have `Theme` management. It is assumed
that page is in `[name]/+page.svelte` file, hence the parameter `name` in the code.

```ts
<script lang="ts">
  import { Theme } from 'freenit'
  import store from '$lib/store'

  const { data } = $props()
</script>

<Theme name={data.name} store={store} />
```

## Themes

The dashboard page for management of `Themes`.

```ts
<script lang="ts">
  import { Themes } from 'freenit'
  import store from '$lib/store'
</script>

<Themes store={store} />
```

## User

The page to add to the dashboard to have `User` management. It is assumed that
page is in `[pk]/+page.svelte` file, hence the parameter `pk` in the code.

```ts
<script lang="ts">
  import { User } from 'freenit'
  import store from '$lib/store'

  const { data } = $props()
</script>

<User pk={data.pk} store={store} />
```

## Users

The dashboard page for management of `Roles`.

```ts
<script lang="ts">
  import { Users } from 'freenit'
  import store from '$lib/store'
</script>

<Users store={store} />
```
