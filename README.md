# AppLink.dev

![BSD-3](https://img.shields.io/badge/License-BSD_3--Clause-blue.svg) ![GitHub Actions Workflow Status](https://github.com/Headbright/applink/actions/workflows/tests.yml/badge.svg)

A simple, privacy respecting service to create a link that redirects to the App Store, Google Play, or Microsoft Store based on the user's platform.

Consider a case where you have an app available on multiple platforms. You want to provide a single link that will redirect the user to the appropriate store based on their platform. AppLink.dev allows you to create such a link.

AppLink.dev is designed to execute entirely in the browser, without any server-side processing. This ensures that no data is collected or stored.

Of course, requesting the page may still expose the visitor's IP address to the server hosting the page.

You're welcome to self-host the page or use the hosted version at [applink.dev](https://applink.dev). Please note that the hosted version may be taken down at any time.

Created with ❤️ by [Konstantin](https://github.com/kkostov).

## Usage

### Simple link

Applink supports parsing a URL with parameters to create a link that will redirect the user to the appropriate store based on their platform. This approach is less flexible that using applink targets (see below).

The following link would redirect a user to their respective app store based on their platform:

```
https://applink.dev/?apple=6444602274&google=org.joinmastodon.android&ms=9ncbcszsjrsb
```

You need to include the platform-specific parameters in the URL:

- For the App Store, include your [app ID](https://developer.apple.com/documentation/storekit/skstoreproductviewcontroller/1502686-loadwithproductid):

`apple=1234`

- For Google Play, include your [app package name](https://developer.android.com/distribute/marketing-tools/linking-to-google-play#OpeningDetails):

`google=package-name`

- For the Microsoft Store, include your [app Store ID](https://learn.microsoft.com/en-us/windows/apps/publish/link-to-your-app):

`ms=app-id`

The same app ID will be used for all App Store platforms (iOS, macOS...).

### Advanced Appplink Targets

A target is a link to a specific app store. A target definition includes one or more filter rules that determine when the target should be used. The filter rules are based on the user's platform, and other properties.

Targets are defined as JSON-LD objects that can be embedded in your website. This approach is more flexible than using a URL with parameters as it allows the inclusion of parameters to each store link as well support for custom app stores.

Example:

```json
{
  "@context": "http://schema.org",
  "@type": "CreativeWorkSeries",
  "hasPart": [
    {
      "@type": "SoftwareApplication",
      "provider": {
        "@type": "Organization",
        "name": "Apple"
      },
      "potentialAction": {
        "@type": "ViewAction",
        "target": {
          "@type": "EntryPoint",
          "urlTemplate": "https://apps.apple.com/app/id12345"
        }
      }
    }
  ]
}
```

- At the root, define a [CreativeWorkSeries](https://schema.org/CreativeWorkSeries) under which you can define the `hasPart` property for each platform.

- Each target is defined as a [SoftwareApplication](https://schema.org/SoftwareApplication) with the `provider` property set to the vendor's name. Currently, we can distinguish between `Apple`, `Google`, and `Microsoft`. More specific operating system based targeting is [currently in the works](https://github.com/Headbright/applink/issues/1).

- To define a url for this target, add a `potentialAction` property with a [ViewAction](https://schema.org/ViewAction) object that has a `url` property.

## Platform detection

Currently, the following platforms are supported:

| User agent platform         | Matching target "provider" | Store           |
| --------------------------- | -------------------------- | --------------- |
| Detected Apple device       | Apple                      | App Store       |
| Detected Google Play device | Google                     | Google Play     |
| Detected Windows device     | Microsoft                  | Microsoft Store |
| Unknown device              | Unknown                    | -               |

Using the `provider` field you can add or more instances of `SoftwareApplication` which will be matched from first to last.

ℹ️ Detection of the browser and platform of the visitor can be quite tricky (especially if we consider older browsers). For now, detection is done explicitly only using the user agent string. This enables us to respect the user's choice if they have opted to modify the user agent string.

## Store Badge

For each target, we will display a badge with the store's logo. Each vendor has specific requirements which may evolve over time so we recommend checking their official guidelines. You can define the image to be shown using the `image` property which is of type [ImageObject](https://schema.org/ImageObject).

```json
{
  "image": {
    "@type": "ImageObject",
    "url": "http://example.com/images/app-store-badge.png",
    "cssStyle": "width: 150px; height: auto;"
  }
}
```

We support the following fields:

- `url` (required): The URL of the image.

- `cssStyle` (optional): The CSS style to append to the `img` tag (it will be added as an attribute like `<img ... style="..." />`)

- `width` (optional): The width of the image in pixels. If provided, we will add it as an attribute to the `img` tag.

- `height` (optional): The height of the image in pixels. If provided, we will add it as an attribute to the `img` tag.

⚠️ Targets without an image tag are ignored and will not be displayed.

## Fallback

If the user's platform is unknown, or none of the provided targets were a match we will display all available app store links with their corresponding badge.

- In case of a URL with parameters, we will provide some default badges for the App Store, Google Play, and Microsoft Store.

- In case of a schema-based target, we will provide a badge for each `SoftwareApplication` object with a defined `image` property.

![Screenshot showing all 3 store badges](/docs/screenshot-all-badges.png)

## Roadmap

Feel free to check the [currently open issues](https://github.com/Headbright/applink/issues). If you have a suggestion or feature request, please open a new ticket 💜!

## Stack

- The component is built using [Vite](https://vitejs.dev) and tests are written using [Vitest](https://vitest.dev).

- The component is written in JavaScript and uses JSDoc for type checking enabled by adding `// @ts-check` at the start of every file. (This was inspired by [Kitten](https://codeberg.org/kitten/app#a-little-less-magic-a-little-more-type)).

- Since this is a static web page, there is no server-side processing.

## Deployment

As a static page, you can deploy this to any static hosting provider as described in the [Vite documentation](https://vitejs.dev/guide/static-deploy.html).
