# 🟣 NgRx Entity Feature Factory

A fully generic, extensible, and scalable pattern for building NgRx stores for entity-based state management.  
This pattern eliminates repetitive code while allowing feature-specific logic via custom reducers, selectors, and effects.

---

## ✅ Features

✔ Auto-generate CRUD Actions  
✔ Auto-generate CRUD Reducer  
✔ Auto-generate Entity Selectors  
✔ Supports `string` or `number` as ID  
✔ Plug-in system for custom reducers  
✔ Compatible with `createFeature()` & `EntityAdapter()`  
✔ Easy integration with Effects  
✔ Ideal for large-scale Angular applications  

---

## 🏗 Folder Structure Example

src/ ├── BaseStore/ │ └── BaseEntityReducer.ts ├── users/ │ ├── users.feature.ts │ ├── users.reducer.ts │ ├── users.selectors.ts │ ├── users.effects.ts │ ├── users.actions.ts (optional) │ └── user.model.ts └── ...

yaml
Copy
Edit

---

## ✅ Factory Usage

### Create a Feature

```ts
export const UsersFeature = createEntityFeature<User, string>('Users', customOns);
Built-in Actions Generated
load()

loadSuccess({ items })

add({ item })

update({ id, changes })

remove({ id })

clear()

✨ Example
Dispatching Actions
ts
Copy
Edit
this.store.dispatch(UsersFeature.actions.load());
this.store.dispatch(UsersFeature.actions.add({ item: user }));
this.store.dispatch(UsersFeature.actions.update({ id: user.id, changes: { name: 'New Name' } }));
this.store.dispatch(UsersFeature.actions.remove({ id: user.id }));
this.store.dispatch(UsersFeature.actions.clear());
Selectors
ts
Copy
Edit
this.store.select(UsersFeature.selectors.selectAll);
this.store.select(UsersFeature.selectors.selectEntities);
this.store.select(UsersFeature.selectors.selectIds);
this.store.select(UsersFeature.selectors.selectTotal);
Custom Actions & Reducers (Feature-specific)
ts
Copy
Edit
export const promoteUser = createAction('[Users] Promote', props<{ userId: string }>());

const customOns = [
    on(promoteUser, (state, { userId }) => {
        const user = state.entities[userId];
        if (!user) return state;
        return UsersFeature.adapter.updateOne({ id: userId, changes: { role: 'Admin' } }, state);
    })
];

export const UsersFeature = createEntityFeature<User, string>('Users', customOns);
Custom Selector Example
ts
Copy
Edit
export const selectFirstUser = createSelector(
    UsersFeature.selectors.selectAll,
    users => users[0] ?? null
);
Example Effect
ts
Copy
Edit
@Injectable()
export class UsersEffects {
    private actions$ = inject(Actions);
    private usersService = inject(UsersService);

    loadUsers$ = createEffect(() =>
        this.actions$.pipe(
            ofType(UsersFeature.actions.load),
            mergeMap(() => this.usersService.getUsers()
                .pipe(
                    map(users => UsersFeature.actions.loadSuccess({ items: users })),
                    catchError(() => of(UsersFeature.actions.clear()))
                )
            )
        )
    );
}
✅ Recommended Patterns
Use customOns to extend reducers safely.

Write feature-specific selectors in users.selectors.ts.

Avoid duplicating CRUD logic, let the factory handle it.

Inject additional non-entity state (e.g., search results) via custom reducer.

✅ Advanced Usage (optional)
Feature	Supported?
Multi-ID types	✅ string / number
Custom Reducers	✅ via customOns
Custom Actions	✅ extend easily
Custom Effects	✅
Custom Selectors	✅
Extra State	✅ via reducer extension
✅ Optional Improvements
Consider extending the factory to also:

Auto-generate common selectors (first, byId, last, total)

Auto-create success + failure actions for load, add, update

Accept extraInitialState to merge feature-specific state without modifying EntityState
