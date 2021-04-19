# NgKeycloak

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 11.2.8.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory. Use the `--prod` flag for a production build.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via [Protractor](http://www.protractortest.org/).

## Keycloak Integration with Angular

### STEP 1

`ng new ng-keycloak` OR
`git clone https://github.com/santoshshinde2012/ng-keycloak.git`

### STEP 2 - Install Required Depedencies

`npm i --save keycloak-js keycloak-angular`

### STEP 3 - Add Keycloak Server Configuration

```
import { KeycloakConfig } from 'keycloak-js';

const keycloakConfig: KeycloakConfig = {
  url: 'http://192.168.64.2:31484/auth',
  realm: 'myrealm',
  clientId: 'demo',
};

export default keycloakConfig;
```

### STEP 4 - Create Auth Module to Handle Keycloak Related Stuff

`ng generate module auth`

### STEP 5 - Keycloak Initialization

```
export function initializer(keycloak: KeycloakService): () => Promise<boolean|Error> {

    const options: KeycloakOptions = {
      config : environment.keycloak,
      loadUserProfileAtStartUp: false,
      initOptions: {
          onLoad: 'login-required',
          checkLoginIframe: true
      },
      bearerExcludedUrls: []
    };

    return (): Promise<boolean|Error> => {
        return new Promise(async (resolve, reject) => {
            try {
                await keycloak.init(options);
                resolve(true);
            } catch (error) {
                reject(error);
            }
        });
    };
}
```
### STEP 6 - Create Auth Service in Auth Module to handle authentication stuff

`ng generate service auth/service/Auth`

```
import { Injectable } from '@angular/core';
import { KeycloakService } from 'keycloak-angular';
import { KeycloakTokenParsed } from 'keycloak-js';

@Injectable()
export class AuthService {

  constructor(private keycloakService: KeycloakService) {}

  public getLoggedUser(): KeycloakTokenParsed | undefined {
    try {
      const userDetails: KeycloakTokenParsed | undefined = this.keycloakService.getKeycloakInstance()
        .idTokenParsed;
      return userDetails;
    } catch (e) {
      console.error("Exception", e);
      return undefined;
    }
  }

  public logout() : void {
    this.keycloakService.logout();
  }

  public redirectToProfile(): void {
    this.keycloakService.getKeycloakInstance().accountManagement();
  }

  public getRoles(): string[] {
    return this.keycloakService.getUserRoles();
  }
}
```

### STEP 7 - Import KeycloakAngularModule and Register Provider KeyloackService in AuthModule

```
import { NgModule, APP_INITIALIZER } from '@angular/core';
import { KeycloakService, KeycloakAngularModule } from 'keycloak-angular';
import { initializer } from './keycloak-initializer';
import { AuthService } from './service/auth.service';

@NgModule({
  declarations: [],
  imports: [KeycloakAngularModule],
  providers: [
    {
        provide: APP_INITIALIZER,
        useFactory: initializer,
        multi: true,
        deps: [KeycloakService]
    },
    AuthService
  ]
})
export class AuthModule { }
```

### STEP 8 - Import AuthModule in main AppModule , So we can use it throught the project

```
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { AuthModule } from './auth/auth.module';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    AuthModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### STEP 9 - Auth Guard configuration

```
import { Injectable } from '@angular/core';
import { ActivatedRouteSnapshot, RouterStateSnapshot, UrlTree, Router } from '@angular/router';
import { KeycloakAuthGuard, KeycloakService } from 'keycloak-angular';

@Injectable()
export class AuthGuard extends KeycloakAuthGuard {

  constructor(protected router: Router, protected keycloakAngular: KeycloakService) {
    super(router, keycloakAngular);
  }

  private isGranted(requiredRoles: string[]): boolean {
    let granted = false;

    if (!requiredRoles || requiredRoles.length === 0) {
      granted = true;
    } else {
      granted = this.roles.some(role => requiredRoles.includes(role));
    }

    return granted;
  }

  public isAccessAllowed(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Promise<boolean | UrlTree> {
    return new Promise(async (resolve, reject) => {

      // If user is not authenticated then needs to redirect to login page
      if (!this.authenticated) {
        this.keycloakAngular.login();
        resolve(false);
        return;
      }

      // If user is not having any role then he will be not granted to access the route/application
      if (!this.isGranted(route.data.roles)) {
        resolve(false);
      }

      resolve(true);

    });
  }

}
```

### STEP 10 - Routing Module Configuration

`ng generate module admin`
`ng generate module user`
`ng generate module admin/admin-routing --flat --module=admin`
`ng generate module admin/user-routing --flat --module=user`
`ng generate component admin/home`
`ng generate component user/home`

