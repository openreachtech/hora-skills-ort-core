---
name: hoc-classes-delegation
description: "How a class uses a delegate — an instance it holds or is handed, such as a client, a repository or a service — and why the call to it sits in a member of its own. Use when writing a class that calls a method of an object it was given or built, especially one whose call can fail and whose failure is reported as a result. How the delegate is built belongs to the dependency-wiring convention; the `invoke~` name, to the method super-prefix convention."
---

# Classes: Delegation

A class delegates when it hands part of its work to another instance and calls that instance's
methods. **The call to the delegate sits in a member of its own, and that member does nothing
else.** What the class does around the call — what it passes on, what it makes of the outcome, a
failure turned into a result included — belongs to the member that calls it.

## What a delegate is

- **A delegate is an instance whose methods the class calls**: one the class holds as a property,
  or one handed to a method as an argument. A client, a repository and a service are the usual
  ones.
- **A call to one of the class's own members is not delegation**, and neither is a call to a
  function the class was handed.

## The delegate is built by a factory method of its own

- How a held delegate is built — the `[TargetClassName]Ctor` getter, the dedicated
  `createXxxx()` factory method, and going through it even where the delegate is built on the
  fly — belongs to `/hoc-wire-dependencies`.
- **It is read here because building and calling are the two seams of one delegation.** The
  factory method is where a test or a subclass swaps the delegate itself; the member below is
  where a test swaps a single call to it.

## The call to the delegate is a member of its own

- **The call sits in a member whose body is the call and nothing more.** It does not transform
  what it passes on, it does not branch on what comes back, and it does not catch what the call
  throws. That member wears the `invoke~` super-prefix — see `/hoc-super-prefix-method-name`.
- **The member that calls it holds everything else.** Building what is passed on, reading what
  comes back, and the `try`/`catch` that turns the delegate's failure into a result all sit
  there.
- **The reason is the test.** With the delegate's call isolated in one member that holds
  nothing else, a test replaces that member to make the delegate succeed, fail or return
  whatever the case needs, and everything the class does around the call is still exercised.
  A call written inline leaves the test only the delegate itself to replace, and every test
  then has to build a fake that fails the right way.

```javascript
// NG: the delegate's call is written inline, inside the member that handles its failure
async sendOrderConfirmation ({
  order,
}) {
  try {
    await this.mailClient.send({
      to: order.email,
      subject: `Order ${order.id} confirmed`,
      body: `Your order ${order.id} has been confirmed.`,
    })

    return this.Ctor.createSendingResult({
      error: null,
    })
  } catch (error) {
    return this.Ctor.createSendingResult({
      error,
    })
  }
}
```

```javascript
// OK: the call is isolated in an invoke~ member, and the public member handles the outcome
import MailClient from './MailClient.js'
import SendingResult from './SendingResult.js'

export default class OrderNotifier {
  constructor ({
    mailClient,
  }) {
    this.mailClient = mailClient
  }

  static create ({
    mailClient = this.createMailClient(),
  } = {}) {
    return new this({
      mailClient,
    })
  }

  static get MailClientCtor () {
    return MailClient
  }

  static get SendingResultCtor () {
    return SendingResult
  }

  static createMailClient () {
    return this.MailClientCtor.create()
  }

  static createSendingResult ({
    error,
  }) {
    return this.SendingResultCtor.create({
      error,
    })
  }

  get Ctor () {
    return /** @type {typeof OrderNotifier} */ (this.constructor)
  }

  /**
   * Send the confirmation of an order.
   *
   * @param {{
   *   order: Order
   * }} params - Parameters.
   * @returns {Promise<SendingResult>} Result of the sending.
   * @public
   */
  async sendOrderConfirmation ({
    order,
  }) {
    try {
      await this.invokeSendMail({
        to: order.email,
        subject: `Order ${order.id} confirmed`,
        body: `Your order ${order.id} has been confirmed.`,
      })

      return this.Ctor.createSendingResult({
        error: null,
      })
    } catch (error) {
      return this.Ctor.createSendingResult({
        error,
      })
    }
  }

  /**
   * Send a mail through the mail client.
   *
   * @param {{
   *   to: string
   *   subject: string
   *   body: string
   * }} params - Parameters.
   * @returns {Promise<void>}
   */
  async invokeSendMail ({
    to,
    subject,
    body,
  }) {
    return this.mailClient.send({
      to,
      subject,
      body,
    })
  }
}
```

- **The `try`/`catch` never moves into the `invoke~` member.** Caught there, the failure is
  decided before the test can replace the call, and the member that should hold what the class
  makes of the outcome is left holding none of it.

## The `invoke~` member is private wherever it can be

- **It is wrapped by the public member that calls it.** It is the seam between the class and its
  delegate, and keeping it off the interface keeps the delegate's call out of what callers can
  couple to.
- **Private here is the member being left out of the published contract**: the public member that
  calls it carries `@public` in its JSDoc (see `/hoc-jsdoc`), and the `invoke~` member does not.
  A native `#` method is never the means — see `/hoc-prohibit-native-features`.

## The outcome is a class, not an object literal

- **What the public member returns for the outcome is an instance of a class**, built through
  its factory method, as `SendingResult` is above. An object literal shaped `{ isSuccess, error }`
  handed back to the caller is an object literal shared across scopes — see
  `/hoc-prohibit-native-features`.
