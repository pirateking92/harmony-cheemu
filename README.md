# Our Winning Submission to Harmony Hackathon 2024

### Challenge
We chose challenge 5: [Harmony Hackathon - Challenge 5](https://github.com/harmonydata/hackathon/blob/main/5-multilingual.md)

I worked on the negation policy. Due to the function of Harmony's app (where medical/psychological forms would be compared for higher-level research and data-usage), a negation system is required in the event a positive and negative question are compared.
For example: "How happy are you?" as a question is the opposite to "how unhappy do you feel on a day-to-day basis?".

##### Edge-cases

Whilst I was going through their current system for the code I noticed that this code block had some edge-cases in english that wouldn't work:

```
def get_change_en(doc) -> dict:
    """
    Identify how to change an English sentence from positive to negative or vice versa.
    :param doc:
    :return:
    """
    for tok in doc:
        if tok.text.lower() in {"always", "rather", "really", "very", "totally", "utterly", "absolutely", "completely",
                                "frequently", "often", "sometimes", "generally", "usually"}:
            return {tok.i: ("replace", "never")}
        if tok.text.lower() in {"never", "not", "n't"}:
            return {tok.i: ("replace", "")}
        if tok.text.lower() in {"cannot"}:
            return {tok.i: ("replace", "can")}
    result = {}
    for tok in doc:
        if tok.text.lower() in {"is", "are", "am", "are", "was", "were", "has", "have", "had"}:
            result[tok.i] = "insert_after", "not"
    if len(result) > 0:
        return result
    #     print ("fallback", doc)
    return {0: ("insert_before", "never")}
```

Specifically:

```
if tok.text.lower() in {"never", "not", "n't"}:
            return {tok.i: ("replace", "")}
```

As contraction negatives are'nt all regular in adding/subtracting the "n't". Therefore the words "can't", "won't" and "shan't" would become "ca", "wo" and "sha" respectively.

To fix this we manually searched for each individual term in the code block to replace it with the correct word:

```
if tok.text.lower() == "ca" and doc[tok.i + 1].text.lower() == "n't":
            return {tok.i: ("replace", "can"), tok.i + 1: ("replace", "")}
        if tok.text.lower() == "wo" and doc[tok.i + 1].text.lower() == "n't":
            return {tok.i: ("replace", "will"), tok.i + 1: ("replace", "")}
        if tok.text.lower() == "sha" and doc[tok.i + 1].text.lower() == "n't":
            return {tok.i: ("replace", "shall"), tok.i + 1: ("replace", "")}
```
This felt like a stable addition as the 'and' operator would target the incorrect change only if it was followed by an "n't" to avoid any typos being changed erroneously.

##### New Languages

Having gone through the code we felt confident to add 4 new languages to the negator policy, French, Spanish, German and Italian using the same system they used for English and Portugese.

See below Italian as the example.

```
def get_change_it(doc) -> dict:
    """
    # Team Cheemu: Identify how to change an Italian sentence from positive to negative or vice versa.
    :param doc:
    :return:
    """
    for tok in doc:
        if tok.text.lower() in {"sempre", "abbastanza", "realmente", "davvero", "veramente", "molto", "molta", "molti", "molte", "totalmente", "assolutamente",
                                "completamente",
                                "frequentemente", "qualche volta", "a volte", "ogni tanto"}:
            return {tok.i: ("replace", "mai")}
        if tok.text.lower() in {"mai", "né", "non", "nessuno", "nulla", "niente"}:
            return {tok.i: ("replace", "")}
    result = {}
    for tok in doc:
        if tok.text.lower() in {"è", "sono", "ero", "erano", "avevano", "avevo", "ho avuto", "sono stato", "sono stata", "sono stati", "siamo stati", "sono state"}:
            result[tok.i] = "insert_before", "non"
    if len(result) > 0:
        return result
    return {0: ("insert_before", "non")}
```

We felt these 4 languages were important in increasing the range of operation for harmony to the Americas and Europe.

These additions would serve as a strong start for the implementation of these languages and we focused strongly on the potential irregularities and differences between the languages' structures.
