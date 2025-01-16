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
            const confirmChange = confirm('{$LANG.paymentmethod}');
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

### Step 3: Update the Language Key

Make sure the language key `$LANG.paymentmethod` is updated or created in your WHMCS language override file. 

1. Navigate to your WHMCS installation path: `whmcsinstallationpath/lang/override/`
2. Open or create the appropriate language file for your default language (e.g., `english.php`).
3. Add or update the following entry:

```php
$_LANG['paymentmethod'] = 'Changing your payment method may cause delays. Do you want to proceed?';
```

You can choose to customize the text or add a new language key. Just ensure it is saved in your language file.

### Step 4: Test Your Changes

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
        <option value="{$defaultpaymentmethod}" selected="selected">[{$LANG.paymentmethoddefault}]</option>
        {foreach from=$paymentmethods item=method}
            <option value="{$method.sysname}"{if $method.sysname eq $defaultpaymentmethod} selected="selected"{/if}>{$method.name}</option>
        {/foreach}
    </select>
    <input type="hidden" name="paymentmethod" value="{$defaultpaymentmethod}" />
</div>
```

This disables the dropdown and ensures the default payment method is retained.


---

## Conclusion

By following these steps, you can:

1. Add a confirmation pop-up for changing payment methods.
2. Disallow changes to the default payment method on the "Account Details" page.

These changes enhance the reliability and security of payment configurations in WHMCS. If you have any questions or run into issues, feel free to contact support or leave a comment.
