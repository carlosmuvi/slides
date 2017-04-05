[comment]: <> (Use reveal-md to show it in slide mode -> https://github.com/webpro/reveal-md)

# Dagger 2

---

# Index

1. ## Pros N Cons
2. Dagger API
 1. Module
 2. Component
 3. Scope
3. Dagger in Marvin
4. Core Changes
5. Migrating existing page w/ tests

---

## Pros N Cons

- Reusable and interchangeable modules
- Separation of concerns (different code paths by environment)
  - mock implementations when testing
  - staging implementations when debugging
  - real implementations in production

---

## Pros N Cons

- Learning curve
- ~~Runtime errors~~ not with Dagger2!

---

## Dagger API

- **`@Module`** + `@Provides`: How to resolve dependencies
- **`@Inject`**: Mechanism for requesting dependencies.
- **`@Component`**: Bridge between modules and classes requesting dependencies.

---

## Module

 - A class whose methods provide dependencies.
 - `@Provides` on each method.
 - `@Module` on the class.


 ```java
 @Module public class CreditCardModule {

     private CreditCardContract.View view;

     public CreditCardModule(CreditCardContract.View view) {
         this.view = view;
     }
     @Provides public CreditCardContract.View view() {
         return view;
     }
     @Provides public CreditCardContract.Presenter providesPresenter(CreditCardPresenter presenter) {
         return presenter;
     }
 }
 ```

---

## Request dependencies

```java
@Inject public CreditCardPresenter(CreditCardContract.View view,
 PaymentMethodUseCase useCase) {
    this.view = view;
    this.useCase = useCase;
}
```

- `@Inject` annotation required
- Constructor injection
  - Add `@Inject` on a single constructor
  - Constructor parameters are dependencies
- Field injection: not recommended, but needed in activities

---

## Components

```java
@Subcomponent(modules = {
  CreditCardModule.class,
  PaymentRepoModule.class,
  LicenseRepoModule.class
}) @ActivityScope public interface CreditCardComponent {
    void inject(CreditCardActivity activity);
}
```

- Bridge between module and injection.
- Where scopes are resolved.
- Injector methods:
 - Receive the class requesting dependencies as parameter
 - When you call it in that class, dagger performs DI.

---

## Dagger in Marvin

<img src="https://docs.google.com/drawings/d/1FWPMKj9_lhYIG68iJF4-Rgs_2o7bsDQhmt9QNXaPpkU/pub?w=800&h=450">

---

## Core changes (1)

- **[`Api` class]** Migrated network Singletons  to Dagger
  - `@Inject` TuroService instead of using `Api#getService`.
  - `Api#getService` deprecated but still working.

---

## Core changes (2)

- **[`Repository`]** Migrated existing repo singletons to Dagger
  - `@Inject` singleton repositories instead of using `Repository#getInstance`
  - [Workaround] Non migrated classes access to singleton repositories via component

```java
ListingDataContract.Repository listingRepository =
    ((BaseActivity) getActivity())
    .getApplicationComponent()
    .listingRepository();

presenter = new ListingAvailabilityPresenter(this,
new ListingUseCase(listingRepository));
```

---

## Migration time!
