---
title: "Obsidian Calendar Import"
date: 2021-11-18T05:30:29-06:00
draft: true
---

Recently, a friend pointed me to a very interesting tool: [Obsidian](https://obsidian.md). Marketed as a "Second-brain", it's primary purpose is to make it easy to take and explore notes. I have found that it not only does an incredible job of that, but it can also do a good job with other things liike documentation.

I've always _tried_ to take notes, especially when it comes to my job - it's one of my primary justifications for having my Surface tablet. In the past I used OneNote (primarily for the ease of handwriting notes if I feel so inclined), but it was difficult to keep up with it the way I feel I should. It had nice integrations, like being able to create a new page specifically for meetings, but linking to other areas of the notebook (or to other notebooks, because why not), and even just organization of notes was always tedious. 

When I picked up Obsidian, I immediately fell in love, and my desire to take good notes started to shine again. It's a tool that not only makes it easy to take notes, but to link to other items and even defer brain-cycle heavy operations like organization until later with what I like to call Note Refactoring. That it's stored in a non-propritary format that's easy to type and read (Markdown), and has the capability of actively querying pages as you would a database is just icing on the cake.

I decided to convert cold-turkey - that is, without porting over anything from my existing OneNote notebook(s), unless absolutely necessary. I wanted to see how this would grow organically without any preconceived notions of organization or content. Not long after I started getting into it, I looked over the official plugins and noticed it has the capability of automatically creating daily notes. I know my friend used it this way, but didn't realize it was a baked-in feature, and immediately turned it on, and started to see what else could be done. A solid baseline of a daily template, visible calendar, and even a panel to aggregate all my outstanding tasks into one view were quickly added. But one thing was missing - I wanted to be able to see what my calendar looked like on each day, and I didn't want to leave Obsidian or the daily notes page to do it.

I found [this article](https://benenewton.medium.com/how-i-automatically-import-my-meetings-into-my-daily-note-template-3d9537b17dea) from Ben Newton pretty quick, and it seems to do exactly what I want. Except... it's only for MacOS. The tool providing the actual calendar data is _only_ available on MacOS, and despite a lot of searching I could not find anything equilivant for Windows. So, after talking it over with other friends and developers, I decided to write my own, but do it in a generic way. I found that [such a feature](https://forum.obsidian.md/t/view-calendar-events-in-daily-notes/1723/21) appears to be in relatively high demand, but nothing that matched my use case - specifically pulling an Active Directory calendar (or really any generic ical calendar), determining the events for the day, and importing relevant information about those events into the daily notes page on creation. Note: I don't care about updates to the calendar over the course of a day; those don't happen often and I figure I can handle those manually, if I do at all.

I started with a relatively simple Node script with the [node-ical](https://www.npmjs.com/package/node-ical) package. After some cleanup, this is what I came up with:

```js
/*
For the most up-to-date version:
https://gist.github.com/kingofzeal/4f4d7e187e2a94335dd5f73a664dcac7
*/

const ical = require('node-ical');
const args = require('yargs')
    .scriptName('calendar-fetch')
    .usage('$0 <url> [args]')
    .command('$0 <url>', 'Parse a given calendar URL', (yargs) => {
        yargs.positional('url', {
            describe: 'URL to fetch the calendar from',
            type: 'string'
        })
        .option('date', {
            describe: 'The date to return events on, in YYY-MM-DD format',
            type: 'string'
        })
        .option('showFreeEvents', {
            describe: 'If set, will show events marked as available',
            type: 'boolean'
        })
    })
    .env('CAL')
    .help()
    .argv;

var inDate = new Date();

if (args.date){
    var passedDate = args.date.match(/\d{4}-\d{2}-\d{2}/i)[0];
    var jsDate = new Date(passedDate);

    inDate.setDate(jsDate.getUTCDate());
    inDate.setFullYear(jsDate.getUTCFullYear());
    inDate.setMonth(jsDate.getUTCMonth());
}

if (!args.url){
    console.error('A URL for the calendar is required');
    return;
}

;(async () => {
    let calendar = await ical.async.fromURL(args.url);

    var allEvents = [];

    var beginRange = new Date(inDate.setHours(0, 0, 0, 0));
    var endRange = new Date(inDate.setHours(23, 59, 59, 999));

    for (var k in calendar) {
        var event = calendar[k]
        
        if (!args.showFreeEvents && event['MICROSOFT-CDO-BUSYSTATUS'] == 'FREE'){
            continue;
        }

        if (event.type === 'VEVENT') {
            var title = event.summary;

            var startDate = event.start;
            var endDate = event.end;

            // Calculate the duration of the event for use with recurring events.
            var duration = event.end.getTime() - event.start.getTime();

            // Simple case - no recurrences, just print out the calendar event.
            if (!event.rrule)
            {
                if (startDate.getTime() >= beginRange.getTime() &&
                    startDate.getTime() <= endRange.getTime()){
                    allEvents.push({
                        event: event,
                        beginTime: event.start,
                        endTime: event.end,
                        title: title
                    });
                }
            }
            // Complicated case - if an RRULE exists, handle multiple recurrences of the event.
            else
            {
                // For recurring events, get the set of event start dates that fall within the range
                // of dates we're looking for.
                var dates = event.rrule.between(beginRange, endRange, false);

                // The "dates" array contains the set of dates within our desired date range range that are valid
                // for the recurrence rule.  *However*, it's possible for us to have a specific recurrence that
                // had its date changed from outside the range to inside the range.  One way to handle this is
                // to add *all* recurrence override entries into the set of dates that we check, and then later
                // filter out any recurrences that don't actually belong within our range.
                if (event.recurrences != undefined)
                {
                    for (var r in event.recurrences)
                    {
                        var recurrenceDate = new Date(r);
                        // Only add dates that weren't already in the range we added from the rrule so that
                        // we don't double-add those events.
                        if (recurrenceDate.getFullYear() === inDate.getFullYear() &&
                            recurrenceDate.getMonth() === inDate.getMonth() &&
                            recurrenceDate.getDate() === inDate.getDate()) {
                                dates.push(recurrenceDate);
                        }
                    }
                }

                // Loop through the set of date entries to see which recurrences should be printed.
                for(var i in dates) {
                    var date = dates[i];
                    var curEvent = event;
                    var showRecurrence = true;
                    var curDuration = duration;

                    startDate.setDate(date.getDate())
                    startDate.setFullYear(date.getFullYear())
                    startDate.setMonth(date.getMonth())

                    // Use just the date of the recurrence to look up overrides and exceptions (i.e. chop off time information)
                    var dateLookupKey = date.toISOString().substring(0, 10);

                    // For each date that we're checking, it's possible that there is a recurrence override for that one day.
                    if ((curEvent.recurrences != undefined) && (curEvent.recurrences[dateLookupKey] != undefined))
                    {
                        // We found an override, so for this recurrence, use a potentially different title, start date, and duration.
                        curEvent = curEvent.recurrences[dateLookupKey];
                        startDate = curEvent.start;
                        curDuration = curEvent.end.getTime() - curEvent.start.getTime();
                    }
                    // If there's no recurrence override, check for an exception date.  Exception dates represent exceptions to the rule.
                    else if ((curEvent.exdate != undefined) && (curEvent.exdate[dateLookupKey] != undefined))
                    {
                        // This date is an exception date, which means we should skip it in the recurrence pattern.
                        showRecurrence = false;
                    }

                    // Set the the title and the end date from either the regular event or the recurrence override.
                    var recurrenceTitle = curEvent.summary;
                    endDate = new Date(startDate.getTime() + curDuration);

                    // If this recurrence ends before the start of the date range, or starts after the end of the date range,
                    // don't process it.
                    if (endDate.getTime() < beginRange.getTime() ||
                        startDate.getTime() > endRange.getTime()) {
                        showRecurrence = false;
                    }

                    if (showRecurrence === true) {
                        allEvents.push({
                            event: event,
                            beginTime: startDate,
                            endTime: endDate,
                            title: recurrenceTitle
                        });
                    }
                }
            }
        }
    }

    allEvents.sort((a, b) => a.beginTime.valueOf() - b.beginTime.valueOf());

    const timeFormatOptions = { hour: '2-digit', minute: '2-digit', hour12: false};

    allEvents.forEach(event => {
        if (event.beginTime.getTime() === event.endTime.getTime()){
            return;
        }

        console.log(`- ${event.beginTime.toLocaleTimeString([], timeFormatOptions)}->${event.endTime.toLocaleTimeString([], timeFormatOptions)} - ${event.title}`);
    });
})().catch(console.error);
```

The bulk of the logic is just handling recurrences, but at the end of the day it outputs a list of all events where I am marked as 'Busy', in cronological order. I created a `.scripts` folder under my Obsidian vault and dropped this in. Because of the npm package requirements, I also added a [`package.json`](https://gist.github.com/kingofzeal/4f4d7e187e2a94335dd5f73a664dcac7#file-scripts-package-json) and installed those packages, but now at the very least I can run this in the command line and just copy/paste the output. I also wanted to make sure a specific date could be specified so I could create new pages in advance or explore my calendar on a day-by-day basis if I needed to (it was also helpful for testing over the weekend).

But this isn't good enough. I want to have this data automatically brought in when the page is created, like the previous article I found. So, I went down the same path they did - using the Templater plugin. The one downside with Templater is that the configured settings (particularly the User Functions) do not syncronize with the vault. In my case, my vault is stored in OneDrive so I can modify it on any of my computers, but I need to set up the User Function on each computer explicitly before this works. I enabled user functions and created a `dailyEvents` function with the following command: `node .scripts/cal.js <calendar_url>`, and also ensured the `Trigger Templater on new file creation` option was enabled. Finally, in my daily template note, I simply added this snippet where I wanted the output: `<% tp.user.dailyEvents({CAL_DATE: tp.file.title }) %>`. 

The name of the file gets passed into the script as an environment variable, which the script then uses as it's date. This means I can create pages in the future, and they will be generated with the _current_ calendar for that day.


I did go down the route of trying to make this into a plugin - I really did. I have some awesome ideas if it were possible, but sadly because Obsidian is an Electron application, it blocks requests to fetch the calendar file due to CORS. It's always CORS. Unfortunately, the workarounds available for this issue are not acceptable for me, so I abandoned the project. If someone else wants to take it up (or has other thoughts on how to make it work), my script should be _easily_ convertable, and I would welcome the more native integration, and could have features like multiple calendars and custom output templating. However, with the script (and dependencies) being syncronized directly with the vault, as long as I have node installed on any computer where I'm generating new daily pages, this works just fine.

I hope this helps others, and if it does leave a note! I will probably watch [the gist](https://gist.github.com/kingofzeal/4f4d7e187e2a94335dd5f73a664dcac7) more than this blog post, but I'm always open to improvements.