# Writeup

> [Repo](https://github.com/questral/data-viz-my-portfolio)

> [Part 1](https://questral.github.io/data-viz-my-portfolio/a3-part1)
> [Part 2](https://questral.github.io/data-viz-my-portfolio/a3-part2)


## Rationale

To be completely transparent, I was limited in my decisions by my lack of experience with d3. However, there were still some overall goals I had in mind. One was simply revealing the main pollutant for each data point; this was done with a simple [color change/shape change] This information is also shown in a tooltip when you hover over the points on the rightmost graph. I also wanted to do something with brushing. I wondered what you could do with brushed data besides expanding it; while it makes it more visible, it doesn’t show new information, or allow for different interpretations. I had the idea to simply sort the selected points, with the goal of letting the viewer look into when the worst AQI days in that range were. To do that, I added a tooltip that appears when you hover over the points, showing the max pollutant and the timestamp of the point.

My earlier ideas were also related to brushing, but I wasn’t sure at first how to provide additional value. One idea was to highlight all the points above the viewer’s cursor and count them, displaying that count as the number of days at or above that AQI. I did like that idea a lot; I felt like it took a visual analysis you could do on your own and made it more shocking by reframing it in that way. However, it didn’t feel like I was doing anything transformative or particularly insightful, so I chose not to pursue that. Another idea I had was to populate a map with the data points (after color coding them) from all of the cities, letting you scrub through the days and possibly identifying patterns of air flow in the way the colors fluctuated. I skipped that idea partially because of my ability, but also because it didn’t feel like there were enough datasets to have a satisfying coverage of color, since there would only be one data point per station for each day.

I also modified my final idea during the process of implementing it. Originally, it was a 2D brush, but I realized there were some issues with how it represented the data. For instance, if there was an outlier above the values you selected, it wouldn’t factor into the rankings, making it seem like it didn’t exist. The ability to make the brush range so specific was effectively dulling its functionality. Because of that, I opted for a 1D brush instead. It would make sure to include all the data points, even if you happened to not see them, keeping viewers from drawing incorrect conclusions.


## Process

The development process was difficult. Starting the project was by far the hardest part; I didn’t know what anything did, and it seemed like the errors had no reason to them. It seemed like there were a lot of different ways of doing the same thing, and I didn’t know how to identify which ways were helpful for me, so there was a long process of trial and error in simply setting up the structure of my chart component. It was also a struggle to figure out how to pass in and manipulate the data. I also hoped Lab 7 would help me with learning Svelte and introduce me to making basic plots, but I got an error early on that I couldn’t fix no matter what I tried. The biggest thing that tripped me up was not knowing where to use which programming language, and a complete lack of familiarity with both Svelte and d3. This was what I struggled with through fall break, and I fully lost confidence in my ability to get it working. However, I was able to enlist Chieri’s help since she’s a CS student (I’m a design student) and since we already knew each other. We were able to put a lot of time into our separate projects, and she was extremely helpful in explaining syntax and data handling mistakes I didn’t have the experience to recognize. With her help I was finally able to make a break through and get a basic d3 tutorial working. Starting from the dataset selection box, I was able to implement more and more complex elements, using the previous ones as foundation for the later ones. From there, it was a process of finding more examples online, extrapolating what I needed to change, and bug hunting to get it to work. (d3-graph-gallery.com is where I got most of my base code.) As I made progress, I noticed I was getting better at reasoning when and where I would need to use different Svelte runes, or how to use d3 functions to make my data easier to work with. And I already have some experience with JavaScript and HTML, so the parts where I could use that were much less painful. But a lot of the process was just staring at code to understand what it did. Overall, I spent at least 30+ hours on this project.


## Conclusion

In my struggle to get this done, I think I lost sight of ways I could make my visualization more effective. With more time, I would have added some redundant encodings; I think it would have been useful to use a shape as well as a color for the main pollutants, and to link size to the severity of the AQI. I also wonder about ways of encoding the timestamp of each point into the mini graph on the right, since at the moment it’s only visible when you hover over it. I think I could potentially do that with some second layer of color. I neglected the PM2.5 rating as well. Finally, zooming and panning would definitely have been useful as quality of life tools.

I did not use AI, but as I mentioned I got a lot of code from d3-graph-gallery.com, as well as Observable HQ, YouTube, and the Svelte and d3 documentation.






<!-- # sv

Everything you need to build a Svelte project, powered by [`sv`](https://github.com/sveltejs/cli).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```sh
# create a new project in the current directory
npx sv create

# create a new project in my-app
npx sv create my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```sh
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```sh
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment. -->
