# Question

Questions (and possible answers) for symptotrack.org.

## Defaults
You can add defaults for questions & groups in the `defaults` directory. Defaults for questions can be based on question type and/or question variant.

Look in the defaults config file for a list of current/supported defaults.

## Basic question config

#### required
You can add `required: true` to a question to make it required

#### required_answer
You can add `required_answer: <answer>` to inputs (all but multiselect & coordinates) to force user to enter a specific answer. This is currently used for forced checkboxes.

#### other
You can add `other: true` to a question to enable submission of string input instead of the questions type based data

#### Conditions
You can add conditions to questions to make them only appear when a specific answer was given to another question.

For instance, to make a question depend on the answer `true` to question `do_you_like_coffee` you could do this:
```
"how_many_cups_did_you_drink": {
  "type": "integer",
  "min": 4,
  "max": Infinity,
  "conditions": [
    {
      "question": "do_you_like_coffee",
      "answer": true
    }
  ]
}
```

For convenience we also support the `not_answer` key.

## Types & variants config
Types define the type of answers (boolean, string, etc). Variants define a variant of a input and enable you to set some defaults for variant only.

#### coordinates
An array with length 2 containing 2 floats (latitude & longitude)

#### bool
A boolean is true or false, `filter` can be passed to indicate that we want to be able to filter by this datapoint

#### integer
A number without decimals

- `min`: minimal value that number can be
- `max`: minimal value that number can be

#### float
A number with decimals

- `min`: minimal value that number can be, with decimals
- `max`: minimal value that number can be, with decimals

#### select
A select question has multiple possible answers but only one answer can be given

- `options`: A list of options

#### text
Well, text

#### date
Well, date

#### multiselect
A multiselect, like select, has multiple possible answers and multiple answers can be passed

- `options`: A list of options


## Questionaires
Each questionaire is defined in a config file in `questionaires`. 

A questionaire is a group of questions:
```
{
  "name": "basic",
  "groups": {
    "group_tag_1": {
      "recurring": true,
      "questions": {
        "question_tag_1": {
        },
        "question_tag_2": {
        }
      }
    }
  }
}
```

When a question group is `recurring`, the questions it contains will be asked again when a respondent returns to update a filled in questionaire.

Questions can be configured like this:
```
"year_of_birth": {
  "type": "integer",
  "min": 1900,
  "max": 2020
},

"fever": {
  "type": "boolean"
}

"fever_degrees": {
  "type": "float",
  "variant": "temperature",
  "conditions": [
    {
      "question": "fever",
      "option": true
    }
  ]
}
```

## Translations
All questions, answers and groups can be translated. you can add files to the `translations` folder in a questionaires folder.

#### Errors
You can translate the following error tags:

- `required`: field is required but no value passed
- `out_of_bounds`: number field received number outside of min/max
- `invalid_option`: (multi)select received value that is not an option (and field did not support `other`)

#### Groups
For every group, you can tranlate:

- `title`: Title of the group
- `next`: Button to continue to next question group

#### Questions
For questions, the question and answers can be translated

- `question`: The actual question
- `answers`: translate the question tags here
- `skip`: if question is not required, translate the skip button
- `other`: translate the "other" option, allowing respondent to input string
- `placeholder`: placeholder for inputs
- `tooltip`
  - `question`: question for tooltip
  - `answer`: actual tooltip description

A translation of a question would look like this:
```
"fever_degrees": {
  "question": "How warm are you? in Celcius"
  "skip": "i don't know",
  "other": "Type your answer"
}
```

## Using this as a module

#### `get_questionaires()`
Returns a array of questionaires found in repository

#### `get_questionaire(name, only_recurring = false)`
Returns a config object for questionaire, with defaults merged in for every question that did not specify some properties

it's possible to pass `only_recurring`, which will only return questiongroups that are "recurring", meaning they have to be filled in on subsequent responses from the same respondent.

#### `get_questionaire_translations(name, locale)`
Returns a config object containing all translations for questionaire, with defaults merged in all unspecified keys

#### `get_error_translations(locale)`
Returns config object containing translation of error tags

#### `validate(questionaire, data)`
Validate a data object based on the questionaire

This function will return a sanitized data object on success (with keys removed that are not part of the questionaire)

It will throw a ValidationError on encountered errors, error tags are contained in the `fields` property of this error object

```
{
  let questionaire = symptotrack.get_questionaire('test');
  try {
    validate(questionaire, {});
  } catch(err) {
    assert.propertyVal(err.fields, 'year_of_birth', 'required');
  }
}
```



