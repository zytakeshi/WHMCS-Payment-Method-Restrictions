# How to Add a Pop-Up and Disallow Users from Changing the Default Payment Method in WHMCS

This guide will walk you through two tasks:

1. Adding a pop-up confirmation when users attempt to change their payment method.
2. Disallowing users from changing their default payment method on the "Account Details" page in WHMCS.

## Adding a Pop-Up Confirmation

You can add a pop-up to confirm changes to the payment method using JavaScript. Here are the steps:

### Step 1: Locate the Relevant Template

Locate the template file where the payment method dropdown is rendered. For invoices, it is usually in the `viewinvoice.tpl` file:

```
/templates/<your-template>/viewinvoice.tpl
```

### Step 2: Add JavaScript for the Pop-Up

In the template file, locate the `paymentmethod` dropdown. Add a script that triggers a confirmation dialog when the user attempts to change their payment method.

Example code:

```html
<select name="gateway" id="paymentMethodDropdown" class="form-control">
    {foreach from=$paymentmethods item=method}
        <option value="{$method.sysname}"{if $method.sysname eq $defaultpaymentmethod} selected="selected"{/if}>{$method.name}</option>
    {/foreach}
</select>

<script>
    document.addEventListener('DOMContentLoaded', function () {
        const dropdown = document.getElementById('paymentMethodDropdown');

        dropdown.addEventListener('change', function (event) {
            const confirmChange = confirm('Changing your payment method may cause delays. Do you want to proceed?');
            if (!confirmChange) {
                // Reset to previous value if the user cancels
                dropdown.value = dropdown.getAttribute('data-default');
            } else {
                dropdown.setAttribute('data-default', dropdown.value);
            }
        });

        // Set initial default value
        dropdown.setAttribute('data-default', dropdown.value);
    });
</script>
```

### Step 3: Test Your Changes

After saving the template file, visit the invoice page and attempt to change the payment method. The pop-up should appear.

---

## Disallowing Users from Changing the Default Payment Method

To disallow changes to the default payment method on the "Account Details" page, you can either disable the dropdown entirely or enforce restrictions using hooks.

### Option 1: Disable the Dropdown

#### Step 1: Locate the Template File

Find the `clientareadetails.tpl` file:

```
/templates/<your-template>/clientareadetails.tpl
```

#### Step 2: Modify the Dropdown

Locate the section rendering the payment method dropdown and disable it:

```html
<div class="form-group">
    <label for="inputPaymentMethod" class="control-label">{$LANG.paymentmethod}</label>
    <select name="paymentmethod" id="inputPaymentMethod" class="form-control" disabled>
        <option value="{$defaultpaymentmethod}" selected="selected">[Default Payment Method]</option>
        {foreach from=$paymentmethods item=method}
            <option value="{$method.sysname}"{if $method.sysname eq $defaultpaymentmethod} selected="selected"{/if}>{$method.name}</option>
        {/foreach}
    </select>
    <input type="hidden" name="paymentmethod" value="{$defaultpaymentmethod}" />
</div>
```

This disables the dropdown and ensures the default payment method is retained.

### Option 2: Use a WHMCS Hook

To enforce restrictions programmatically, create a hook:

#### Step 1: Create the Hook File

Create a new file in `/includes/hooks/` named `restrict_payment_method.php`.

#### Step 2: Add the Hook Code

```php
<?php

use WHMCS\Database\Capsule;

add_hook('ClientEdit', 1, function ($vars) {
    $clientId = $vars['userid'];
    $originalPaymentMethod = Capsule::table('tblclients')
        ->where('id', $clientId)
        ->value('defaultgateway');

    if (isset($vars['paymentmethod']) && $vars['paymentmethod'] !== $originalPaymentMethod) {
        throw new \WHMCS\Exception\Fatal('You cannot change your default payment method. Please contact support for assistance.');
    }
});
```

This hook ensures the default payment method cannot be changed via the client area or API.

### Step 3: Test Your Hook

Save the file and attempt to update the payment method on the "Account Details" page. The system should block any changes and display the error message.

---

## Conclusion

By following these steps, you can:

1. Add a confirmation pop-up for changing payment methods.
2. Disallow changes to the default payment method on the "Account Details" page.

These changes enhance the reliability and security of payment configurations in WHMCS. If you have any questions or run into issues, feel free to contact support or leave a comment.
