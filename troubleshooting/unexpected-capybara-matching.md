# Unexpected Capybara Matching

If you just use things like `click_on '2'` or `must_have_content '111'` in your tests, they might fail in weird ways, or worse, pass when they should fail.

This is because capybara is looking in the whole page, and the string you specify could easily match something elsewhere on the page, like the text of an error message or a user's name from our factories ("First Name 254 Last Name 254"). Also note that by default, capybara will match a substring, so `click_link("Password")` would also match a link that says "Password confirmation".

Better is to either use `within` or otherwise specify a narrower CSS selector. The two above could be rewritten, potentially, as:

```
within '.pagination' do
  click_on '2'
end
```

and

```
must_have_selector '.item_number', text: '111'
```
