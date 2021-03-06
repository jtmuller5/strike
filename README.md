<!-- 
This README describes the package. If you publish this package to pub.dev,
this README's contents appear on the landing page for your package.

For information about how to write a good package README, see the guide for
[writing package pages](https://dart.dev/guides/libraries/writing-package-pages). 

For general information about developing packages, see the Dart guide for
[creating packages](https://dart.dev/guides/libraries/create-library-packages)
and the Flutter guide for
[developing packages and plugins](https://flutter.dev/developing-packages). 
-->

This package is a pure Dart SDK wrapper for the Strike APIs ([offical docs](https://docs.strike.me/)).

Strike APIs enable you to accept payments securely and integrate your app with [Strike](https://invite.strike.me/WYJFFO).

## Support
<a href="https://buymeacoffee.com/mullr" target="_blank"><img align="left" src="https://cdn.buymeacoffee.com/buttons/default-orange.png" alt="Buy Me A Coffee" height="41" width="174"></a>
<br>
<br>

<a href="https://keepmylightson.xyz/support/joemuller" target="_blank"><img align="left" src="https://cdn.jsdelivr.net/gh/jtmuller5/strike/socials/Keep My Lights On BWY.png" alt="Keep My Lights On" height="40" width="200"></a>
<br>
<br>

[:heart: Sponsor on GitHub](https://github.com/sponsors/jtmuller5) 


## Roadmap

- [x] Find user profiles by handle
- [x] Find user profiles by ID
- [x] Issue basic invoice 
- [x] Issue invoice to specific receiver
- [x] Find invoice by ID
- [x] Issue quote for invoice
- [x] Cancel unpaid invoice
- [x] Get currency exchange rates
- [x] Open Strike App from Invoice (mobile)
- [x] Open Strike App from Quote (mobile)
- [ ] Open Strike App from Invoice (web)
- [ ] Open Strike App from Quote (web)
- [ ] Get webhook events
- [ ] Find webhook events by ID
- [ ] Get webhook subscriptions
- [ ] Create new webhook subscriptions
- [ ] Find webhook subscription by ID
- [ ] Update webhook subscription
- [ ] Delete webhook subscription

## Getting started

In order to use the Strike API, you will need to [request an API key](https://developer.strike.me/en/).

## Secure your API Key
You can use the [flutter_dotenv](https://pub.dev/packages/flutter_dotenv "flutter_dotenv") package to keep your API key safely out of public repositories.
1. Add flutter_dotenv to your pubspec.yaml
2. Add \*.env to your project's .gitignore file
3. Create a .env file in the root of your project and add the following contents
```yaml
STRIKE_API_KEY=<YOUR_API_KEY>
```
4. Add the .env file to the assets section of your pubspec.yaml and run flutter pub get
```yaml
assets:
  - .env
```
## Usage
Create your Strike instance.

Without flutter_dotenv:

```dart
Strike _strike = Strike(apiKey: '<YOUR_API_KEY>');
```
With flutter_dotenv:

```dart
await dotenv.load(fileName: '.env');

Strike _strike = Strike(apiKey:dotenv.env['STRIKE_API_KEY']!);
```

## Issue an Invoice
The only thing *required* to issue an invoice is an InvoiceAmount (which includes the quantity and type of currency being requested). All other fields are optional.

Both of the following methods will return the generated Invoice.

### Issue an Invoice for Yourself
```dart
await strike.issueInvoice(
  handle: null,
  correlationId: null,
  description: null,
  invoiceAmount: InvoiceAmount(
     amount: 10,
     currency: CurrencyType.USD,
  ),
);
```
When you issue an invoice without specifying a receiver, the invoice is created with your own personal Strike ID as both the "Issuer" and the "Receiver". In other words, when this invoice is paid, you receive the funds.

### Issue an Invoice for Someone Else
```dart
await strike.issueInvoice(
  handle: '<RECEIVER_HANDLE>',
  correlationId: null,
  description: "Nice work!",
  invoiceAmount: InvoiceAmount(
     amount: 1,
     currency: CurrencyType.BTC,
  ),
);
```

### Open an Invoice in Strike

The Strike mobile app accepts deep links of the following format: https://strike.me/pay/<INVOICE_ID>

Below is the workflow for using the link:
1. User opens the link
2. User see's the invoice's description and amount
3. User presses "Pay"
4. The Strike app generates a QR code for the invoice

This package depends on [url_launcher](https://pub.dev/packages/url_launcher). Each Invoice has an openStrikeApp() method that will open the appropriate deep link.

```dart
OutlinedButton(
  child: const Text('Open Strike App'),
  onPressed: () {
   invoice?.openStrikeApp(); // launchUrl(Uri.parse('https://strike.me/pay/$invoiceId'));
  },
),
```

If you'd rather have the Strike app handle the QR code generation, you can ignore the "Issue a Quote" section below.

### Cancel an Unpaid Invoice
```dart
Invoice? cancelledInvoice = await strike.cancelUnpaidInvoice(invoiceId: invoice?.invoiceId);
```

This endpoint will respond with a 422 error code if the Invoice is already cancelled.

## Issue a Quote
Once you have an Invoice, you can generate a quote for it ([source](https://docs.strike.me/use-cases/tipping-platform)).

![Invoice to Quote](https://docs.strike.me/assets/images/tipping-diagram@2x-3505c72116e5aaa4b55682fdccafb4db.png)

```dart
strike.issueQuoteForInvoice(invoiceId: invoice.invoiceId)
```

### Open a Quote in Strike

The URL scheme for lightning payments is "lightning:<LIGHTNING_INVOICE>"

All apps that can accept lightning payment requests can be opened with this URL scheme.

```dart
OutlinedButton(
  child: const Text('Open Strike'),
  onPressed: () {
   quote.openStrikeApp(); // launchUrl(Uri.parse('lightning:$lnInvoice'));
  },
),
```

At the moment, the Strike browser extension cannot be opened by using launchUrl(). Instead, you will need to use the Link widget from the url_launcher package.

```dart
Link(
    target: LinkTarget.blank,
    uri: Uri.parse('lightning:<LNINVOICE>'),
    builder: (context, onTap) {
      return ElevatedButton(
        onPressed: onTap,
        child: const Text('Send'),
      );
    },
  );
```

## Generate a QR Code for the Quote
Each Quote contains an "lnInvoice" field which is the encoded lightning invoice. Using the [qr_flutter](https://pub.dev/packages/qr_flutter) package you can easily turn that field into a QR Code your users can scan.

```dart
if(quote.lnInvoice != null) {
  return QrImage(
    data: quote.lnInvoice!,
    version: QrVersions.auto,
    size: 200.0,
  );
}
```
Note that the generated QR code must be scanned from inside the Strike app. If you scan it outside of the app, it simply opens your note app with the QR contents.

## Relevant Issues
- [https://github.com/flutter/flutter/issues/43809](https://github.com/flutter/flutter/issues/43809)
- [https://github.com/flutter/flutter/issues/102608#issuecomment-1110832339](https://github.com/flutter/flutter/issues/102608#issuecomment-1110832339)
