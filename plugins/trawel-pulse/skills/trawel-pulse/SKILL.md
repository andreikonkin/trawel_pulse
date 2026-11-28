---
name: trawel-pulse
description: Keep the user's Trawel Pulse travel account and answer from it - add trips from tickets, emails or a statement, count days by country, tax residence, how long more one may stay, personal limits of days abroad, citizenship goals and visas, and cards of the year, a goal or a calendar to share; research citizenship by residence programmes (conditions, dates needed, an estimate of when one may apply); and answer travel questions with its public tools (a country's safety, visa rules, emergency numbers, natural disasters, recent changes, cheapest flights). Use when the user talks about their trips, days abroad, tax residence, residence permits, citizenship by residence, visas, travel budgets, sharing their year in travel, or travel safety and flights.
---

# Trawel Pulse

The Trawel Pulse tools keep the user's travel account (the same as on trawelpulse.com) and count from it. Answer in the user's language.

## The account

- Start with `get_profile` and `list_moves` to see what's there before changing anything.
- A trip is entered as its **moves**: one per border crossing, there and back, in order (`add_moves` takes many at once). A move has the day left, the day arrived if not the same (a night flight), the country left and the one arrived in; cities and a note (a flight number) are optional. Between moves the user is in the country of the last arrival.
- From tickets, booking emails or a bank statement: list the moves you read and add them in one call; say which ones you weren't sure of. Don't invent dates.
- Moves to come (up to a year ahead) are plans: they count to the end of the year and in the goals, and warn when they'd lose a tax residence.
- Citizenships, tax residence, visas and citizenship goals are in the profile: `update_profile`, and the visa and goal tools.
- For days and limits use `get_days`: the days by country as each country counts them (tax and stay days), tax residence, how long more the user may stay, and the goals' progress.
- To show them as the site draws them, use the views: `show_goals`, `show_tax_residence`, `show_days`, `show_visited_countries` (a world map of the countries been to, with their flags), and `show_calendar` (a year's calendar of where they were each day: the year at a glance, a row a month with the trips as bars, or each month's days with `layout: months`; the view switches between them and between the years with days).
- Family and friends may open their accounts to the user on the site (`list_people`): pass `person` (their id or name) to any account tool to look at theirs, or change it if they allowed; say whose account it is.

## Personal limits of days abroad

- The user's own budgets by country and year, apart from any law: at most so many days abroad in the year, and in one trip. They're in the profile (`travel_limits`, from `get_profile`).
- `set_travel_limit` replaces both budgets of a country and year: carry over the one the user keeps, or it's cleared. Zero is a real limit of no days; without either budget the setting is removed.
- `get_days` gives them counted under `travel_planning`: the days so far and planned, those left, and the overruns of the year and of a trip.
- They never change the tax, visa or citizenship counts.

## Citizenship by residence

- `list_citizenship_programs` lists the programmes (their ids, names and filters); `get_citizenship_program` gives one's conditions, the dates it needs, citizenship at birth for children born there, life after citizenship, and the official sources with the day they were checked.
- `calculate_citizenship` is the estimate of the site's calculator (/citizenship): the days credited, the progress and about when one may apply, from a residence start, without the account or its trips.
- A programme may need dates besides the residence start (marriage, a spouse's citizenship, permanent residence): ask for them; without them the estimate stays pending. Don't invent them.
- It's a time-and-days estimate, not a decision on eligibility: language, exams, status and the other conditions are checked apart. Point to the official sources.
- To follow a programme by the user's real trips, save it as a goal (`add_goal` with its id as `preset`, `since` and the `dates`) when they want to; then `get_days` or `show_goals`.

## Cards to share

- `show_card` shows the pictures the account makes for posts and stories: a year's travels (`year`) or all the years' (`all`), a goal's progress (`goal`, by its place in the list), or a year's calendar (`calendar`). Only the user's own.
- It uses the card's public link, the same as Share in the account: tell the user anyone with the link sees the card (countries and days, no dates, notes or names), and that they turn it off in the account.

## Travel questions

Use the public tools for anyone, signed in or not: `country_safety`, `countries_safety`, `visa_requirements`, `emergency_numbers`, `natural_disasters`, `recent_changes`, `find_city` and `cheapest_flights`. Say the data is aggregated from official sources and may lag; for a decision, point to the official advisory.

## Care

- Changes are real: confirm before deleting a move, a visa or a goal.
- The counts are a guide, not tax or legal advice; say so when the user is close to a limit.
