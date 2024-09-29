---
date: 2024-08-12
tags:
  - book
hubs:
  - visualization
---
# Everday Data Visualization
Authors: Desiree Abbott

# Index
- Storytelling
- Color theory
- Good dashboards
- Good charts

# Storytelling
- The goal is to tell a story.
- Contrasting colors and font styles are recognized in our brains before we even realize it. Use this to highlight the important information

# Color theory
- RGB colors are represented by three values, each 1 byte (8 bits), i.e. 2^8 = 256. This can be represented as three base 10 integers - as long as they are bounded from 0-255, or as two unbounded hexadecimal numbers (16^2 = 256)
- HSL (hue, saturation, lightness) is more natural way of working with colors. The hues are laid out on a color wheel, and adjusting them feels natural. Fixed hues can be adjusted by saturation and lightness as well in order to change their quality.
- [Color palette generator from Google](https://artsexperiments.withgoogle.com/artpalette/) will generate one from an image uploaded
- [Color picker from learnui.design](https://www.learnui.design/tools/data-color-picker.html) will generate various palettes based on seed colors, it's absolutely incredible
- [Gradient generator from learnui.design](https://www.learnui.design/tools/gradient-generator.html) is amazing as well for CSS gradients

# Good dashboards
- Apply gestalt principles and pre-attentive attributes of color, form, spatial position, proximity and symmetry to our text to create a hierarchy of importance, e.g. provide high-level context, highlight important information, minimize supportive information
- Use position to associate related elements. Use space to dissociate.
- Keep font styles readable and simple, e.g. one font, 3 size/color variations, like this [NHS Hospital Tableau report by Pradeep Kumar](https://public.tableau.com/app/profile/pradeepkumar.g/viz/NHSHospitalAdmittedPatientCareActivity/viz)

# Good charts
- Goal / purpose
	- A good chart is simple.
	- Always tell a story.
	- A good chart holds the users attention.
	- Ultimately, a good chart gets the point across.
- Details
	- Declutter grid lines, axes, labels, etc.. Only keep what you need.
	- Stacked bar charts suck. Just unstack them.
	- Labeling bars with simplified metrics is good.
	- Human eye is bad at estimating angles, so keep pie chat segments to a minimum (2-3).
	- Un-spaghetti (detangle) line charts: rather than multiple series on the same chart, use separate charts for each and stack together (vertically, or in grid). Make the x and y-axes consistent. 
- Interactive
	- The most important information should be visible with no interaction required.
	- Use familiar interaction patterns and keep it simple.
	- Design interactivity around how users naturally think about a problem.
