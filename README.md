# WooApi
Woocommerce API service with angular

## Dependency
- HttpClientModule
import { HttpClientModule } from '@angular/common/http';


## Supports woocommerce API version
- Supports V2 version: base url https://example.com/wp-json/wc/v2
- Please see `environment` constact and interceptor for frontend setup


Tested with https protocol. Wordpress version 4.9.6 and WooCommerce version 3.4.1

## Setup instruction
- [Backend setup](#backend-setup-instruction)
- [Frontend setup](#frontend-setup-instruction)

## Api Doc
- https://angular-studio.github.io/wooApi
- https://angular-studio.github.io/wooApi/v5

## Backend Setup instruction

### Enable CORS
Add this code in function.php

```
add_action( 'init', 'nt_cors_enable' );

function nt_cors_enable() {
  header("Access-Control-Allow-Origin: " . get_http_origin());
  header("Access-Control-Allow-Methods: POST, GET, OPTIONS, PUT, DELETE");
  header("Access-Control-Allow-Credentials: true");
  header("Access-Control-Allow-Headers: Authorization, Content-Type");
  header("Access-Control-Expose-Headers: x-wc-totalpages, x-wc-total", false);
  if ( 'OPTIONS' == $_SERVER['REQUEST_METHOD'] ) {
    status_header(200);
    exit();
  }
}
```

### Auth service dependency
- Install JSON API user https://wordpress.org/plugins/json-api-user/, https://wordpress.org/plugins/json-api/
- Install JWT support https://wordpress.org/plugins/jwt-authentication-for-wp-rest-api/

After installatiion activate user controller from JSON-API settings. Under settings > JSON-API > User > activate.



## Frontend Setup instruction

- `yarn add ngx-wooapi@5.x.x` or `npm install --save ngx-wooapi@5.x.x` for angular 5 support
- `yarn add ngx-wooapi@6.x.x` or `npm install --save ngx-wooapi@6.x.x` for angular 6 support
-
- Add interceptor

```
import {
  Injectable,
  // Injector
 } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
  HttpErrorResponse,
} from '@angular/common/http';
// import { Router } from '@angular/router';
import { catchError } from 'rxjs/operators';
import { Observable, throwError } from 'rxjs';

// import { AuthService } from './auth.service';
import { environment } from '../environments/environment';

@Injectable()
export class AppInterceptor implements HttpInterceptor {

  constructor(
    // private injector: Injector,
    // private router: Router
  ) { }

  private includeWooAuth(url) {
    const wooAuth = `consumer_key=${environment.woocommerce.consumer_key}&consumer_secret=${environment.woocommerce.consumer_secret}`;
    const hasQuery = url.includes('?');
    let return_url = '';
    if (hasQuery) {
      return_url =  wooAuth;
    } else {
      return_url = '?' + wooAuth;
    }
    return return_url;
  }

  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    let authRequest;
    // const auth = this.injector.get(AuthService);
    let requestUrl = '';
    if (request.url.includes('api') || request.url.includes('jwt')) {
      requestUrl = `${environment.origin}/${request.url}`;
    } else {
      requestUrl = `${environment.origin}${environment.wcEndpoint}/${request.url}${this.includeWooAuth(request.url)}`;
    }
    authRequest = request.clone({
      url: requestUrl
    });

    return next.handle(authRequest)
      .pipe(
        catchError(err => {
          if (err instanceof HttpErrorResponse && err.status === 0) {
            console.log('Check Your Internet Connection And Try again Later');
          } else if (err instanceof HttpErrorResponse && err.status === 401) {
            // auth.setToken(null);
            // this.router.navigate(['/', 'login']);
          }
          return throwError(err);
        })
      );
  }
}

```

Add this code in your app.module.ts

```
 providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AppInterceptor,
      multi: true
    }
 ]

```
Add new property in environment.ts for angular webapp

```
export const environment = {
  origin: 'https://example.com/appwoo',
  wcEndpoint: '/wp-json/wc/v2',
  woocommerce: {
    consumer_key:  'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    consumer_secret: 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
  }
};
```

Add new providers in app.module
```
import {
  WoocommerceProductsService,
  WoocommerceHelperService
} from 'ngx-wooapi';
```
Add these providers in providers array

```
providers: [
  WoocommerceProductsService,
  WoocommerceHelperService
]
```

Now use it in component

```
import {
  WoocommerceProductsService
} from 'ngx-wooapi';

constructor(
    private wooProducs: WoocommerceProductsService
  ) { }

  ngOnInit() {
    this.wooProducs.retrieveProducts().subscribe(response => {
      console.log(response);
    }, err => {
      console.log(err);
    });
  }

```

All done have fun :)

### Woocommerce Rest API doc
https://woocommerce.github.io/woocommerce-rest-api-docs/

### Progress
- coupons [x]
- customers [x]
- orders [x]
- order-notes [x]
- refunds [x]
- products [x]
- product-variations [x]
- product-attributes [x]
- product-attribute-terms []
- product-categories [x]
- product-shipping-classes []
- product-tags [x]
- product-reviews [x]
- reports []
- tax-rates []
- tax-classes []
- webhooks []
- settings []
- setting-options []
- payment-gateways []
- shipping-zones []
- shipping-zone-locations []
- shipping-zone-methods []
- shipping-methods []
- system-status []
- system-status-tools []
- data []