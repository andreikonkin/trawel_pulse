---
name: trawel-pulse
description: Keep the user's Trawel Pulse travel account and answer from it - add trips from tickets, emails or a statement, count days by country, tax residence, how long more one may stay, citizenship goals and visas; and answer travel questions with its public tools (a country's safety, visa rules, emergency numbers, natural disasters, recent changes, cheapest flights). Use when the user talks about their trips, days abroad, tax residence, residence permits, citizenship by residence, visas, or travel safety and flights.
---

# Trawel Pulse

The Trawel Pulse tools keep the user's travel account (the same as on trawelpulse.com) and count from it. Answer in the user's language.

## The account

- Start with `get_profile` and `list_moves` to see what's there before changing anything.
- A trip is entered as its **moves**: one per border crossing, there and back, in order (`add_moves` takes many at once). A move has the day left, the day arrived if not the same (a night flight), the country left and the one arrived in; cities and a note (a flight number) are optional. Between moves the user is in the country of the last arrival.
- From tickets, booking emails or a bank statement: list the moves you read and add them in one call; say which ones you weren't sure of. Don't invent dates.
- Moves to come (up to a year ahead) are plans: they count to the end of the year and in the goals, and warn when they'd lose a tax residence.
- Citizenships, tax residence, visas and citizenship goals are in the profile: `update_profile`, and the visa and goal tools.
- For days and limits use `get_days`: the days by country as each country counts them (tax and stay days), tax residence, how long more the user may stay, and the goals' progress. Its views draw the same charts as the site.

## Travel questions

Use the public tools for anyone, signed in or not: `country_safety`, `countries_safety`, `visa_requirements`, `emergency_numbers`, `natural_disasters`, `recent_changes`, `find_city` and `cheapest_flights`. Say the data is aggregated from official sources and may lag; for a decision, point to the official advisory.

## Care

- Changes are real: confirm before deleting a move, a visa or a goal.
- The counts are a guide, not tax or legal advice; say so when the user is close to a limit.
