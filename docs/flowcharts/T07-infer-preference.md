# T07 - inferPreference(session, links, txs, products)

Input: session, identity links, valid transactions (excluding GUEST), product catalogue.
Output: Preference (branch, category, brand, budget).

[T07-infer-preference.png]

Pseudocode:

    IF isIdentified AND hasPurchaseHistory THEN
        pref.branch = 'a'
        pref.category = mostFrequentCategory(bought)
        pref.brand = mostFrequentBrand(bought)
        pref.budget = maxUnitPrice(bought) * 120 / 100
    ELSE IF hasViewEvents THEN
        pref.branch = 'b'
        pref.category = mostFrequentCategory(viewed)
        pref.brand = mostFrequentBrand(viewed)
        pref.budget = maxPrice(viewed)
    ELSE
        pref.branch = 'c'
        pref.category = ""
        pref.brand = ""
        pref.budget = maxPrice(products)
    RETURN pref
