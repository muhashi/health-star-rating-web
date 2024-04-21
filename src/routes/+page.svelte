<script>
    import { onMount } from "svelte";
    import "carbon-components-svelte/css/all.css";
    import { Attributes, Category, calculateHealthStarRating, Category1, Category1D, Category2, Category2D, Category3, Category3D } from 'health-star-rating';
    import {
        Form,
        FormGroup,
        NumberInput,
        OutboundLink,
        Select,
        SelectItem,
        Checkbox,
        DataTable,
        Accordion, AccordionItem, UnorderedList, ListItem,
    } from "carbon-components-svelte";
    import StarRating from 'svelte-star-rating';

  
    // theme = "white" | "g10" | "g80" | "g90" | "g100"
    function updateTheme(theme) {
        document.documentElement.setAttribute("theme", theme);
    }

    onMount(() => {
        updateTheme("white");
    });

    const selectItemtoFoodCategoryMap = {
        "general-food": Category.OtherFoods,
        "dairy-beverage": Category.DairyBeverages,
        "dairy-food": Category.DairyFoods,
        "oil": Category.FatsOilsAndSpreads,
        "cheese": Category.Cheese,
        "water": Category.PlainWater,
        "flavoured-water": Category.UnsweetenedFlavouredWater,
        "fruit-and-vegetables": Category.UnprocessedFruitAndVegetables,
        "beverage": Category.NonDairyBeverages,
        "jellies": Category.Jellies,
        "ice-confection": Category.WaterBasedIcedConfection,
    };

    const selectItemtoFoodNameMap = {
        "general-food": "General Food",
        "beverage": "Non-Dairy Beverages",
        "dairy-beverage": "Dairy Beverage",
        "dairy-food": "Dairy Food",
        "oil": "Oils, Fats, and Fat/Oil Spreads",
        "cheese": "Cheese",
        "water": "Plain Water",
        "flavoured-water": "Unsweetened Flavoured Water",
        "fruit-and-vegetables": "Fruit and Vegetables",
        "jellies": "Jellies",
        "ice-confection": "Water-Based Ice Confections"
    };


    const foodCategoryNotes = {
        "general-food": "<strong>General Food</strong> includes all foods that do not fit into any of the other categories. If the food fits into one of the other categories, use that category instead.",
        "dairy-beverage": "",
        "dairy-food": "<strong>Dairy Food</strong> does not include ice cream or alternatives derived from cereals, nuts or seeds. These products fall in General Food.",
        "oil": "<strong>Oils, Fats, and Fat/Oil Spreads</strong> also includes: edible oil, edible oil spreads, margarine, butter.",
        "cheese": "",
        "water": "",
        "flavoured-water": "<strong>Unsweetened Flavoured Water</strong> may only include carbon dioxide, flavouring substances (excluding sugar/sweeteners), mineral salts, safety/stability additives. Waters in this category must not include added sugars, sweeteners, colours, sodium, caffeine, quinine, or any other ingredient that contains energy.",
        "fruit-and-vegetables": "<strong>Fruit and Vegetables</strong> incudes all whole fresh fruit (except coconut) and vegetables, fungi and legumes (except peanuts) as sold with no processing, plus these same products that have only been peeled, cut and/or surface treated and/or blanched and/or frozen (not dried), or canned without the addition of fat, sugars/sweeteners or salt. Excludes canned fruit and vegetables in juice and brine.",
        "beverage": "",
        "jellies": "",
        "ice-confection": "",
    };

    let foodType = null;
    $: category = selectItemtoFoodCategoryMap[foodType] ?? null;
    let energy = 0;
    let sodium = 0;
    let saturatedFat = 0;
    let sugar = 0;
    let percentageFruitVegetableNutLegumeContent = null;
    let fibre = null;
    let protein = null;
    let containsFruitOrVegetable = null;
    let containsNutsOrLegumes = null;
    let healthStarRating = null;

    // just assume if FVNL content but no FVNL checkbox is ticked, that it contains fruit or veges
    $: shouldContainFruitOrVegetable = percentageFruitVegetableNutLegumeContent > 0 && !containsNutsOrLegumes;
    let attributes = [];
    
    $: if (containsFruitOrVegetable || shouldContainFruitOrVegetable || containsNutsOrLegumes) {
        attributes = [];

        if (containsFruitOrVegetable || shouldContainFruitOrVegetable) {
            attributes.push(Attributes.ContainsFruitOrVegetable);
        }

        if (containsNutsOrLegumes) {
            attributes.push(Attributes.ContainsNutsOrLegumes);
        }
    } else {
        attributes = [];
    }

    function onFormChange() {
        const nutritionalInformation = {
            energykJ: energy,
            sodiumMilligrams: sodium,
            saturatedFatGrams: saturatedFat,
            totalSugarsGrams: sugar,
            ...(percentageFruitVegetableNutLegumeContent !== null && { percentageFruitVegetableNutLegume: percentageFruitVegetableNutLegumeContent }),
            ...(fibre !== null && { fibreGrams: fibre }),
            ...(protein !== null && { proteinGrams: protein }),
            attributes,
        };

        try {
            healthStarRating = calculateHealthStarRating(category, nutritionalInformation);
        } catch (e) {
            healthStarRating = null;
        }
    }

    /**
     * Baseline Points
     */
    const commonEnergyRange = [335, 670, 1005, 1340, 1675, 2010, 2345, 2680, 3015, 3350, 3685];
    const commonSodiumRange = [90, 180, 270, 360, 450, 540, 630, 720, 810, 900, 990, 1080, 1170, 1260, 1350, 1440, 1530, 1620, 1710, 1800, 1890, 1980, 2070, 2160, 2250, 2340, 2430, 2520, 2610, 2700];

    let energyRange = commonEnergyRange;
    let sodiumRange = commonSodiumRange;
    let saturatedFatGramsRange = [];
    let totalSugarsGramsRange = [];

    $: if (Category1D(category) || Category2(category) || Category2D(category)) {
        energyRange = [...commonEnergyRange];
        saturatedFatGramsRange = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11.2, 12.5, 13.9, 15.5, 17.3, 19.3, 21.6, 24.1, 26.9, 30, 33.5, 37.4, 41.7, 46.6, 52, 58, 64.7, 72.3, 80.6, 90];
        totalSugarsGramsRange = [5, 8.9, 12.8, 16.8, 20.7, 24.6, 28.5, 32.4, 36.3, 40.3, 44.2, 48.1, 52, 55.9, 59.8, 63.8, 67.7, 71.6, 75.5, 79.4, 83.3, 87.3, 91.2, 95.1, 99];
        sodiumRange = [...commonSodiumRange];
    } else if (Category3(category) || Category3D(category)) {
        energyRange = [...commonEnergyRange];
        saturatedFatGramsRange = [...Array(30).keys()].map(x => x + 1); // [1..30]
        totalSugarsGramsRange = [5, 9, 13.5, 18, 22.5, 27, 31, 36, 40, 45];
        sodiumRange = [...commonSodiumRange];
    } else if (Category1(category)) {
        energyRange = [-1, 31, 61, 91, 121, 151, 181, 211, 241, 271];
        totalSugarsGramsRange = [0.1, 1.6, 3.1, 4.6, 6.1, 7.6, 9.1, 10.6, 12.1, 13.6];
        saturatedFatGramsRange = [Infinity];
        sodiumRange = [Infinity];
    }

    $: baselineData = Array.from({length: 1 + Math.max(energyRange.length, sodiumRange.length, saturatedFatGramsRange.length, totalSugarsGramsRange.length)}, () => ({}))
        .map((_, i) => {
            return {
                energy: energyRange[i] ? ("≤" + energyRange[i]) :
                    (i === energyRange.length ? ">" + energyRange[i - 1] : ""),
                sodium: sodiumRange[i] ? ("≤" + sodiumRange[i]) :
                    (i === sodiumRange.length ? ">" + sodiumRange[i - 1] : ""),
                saturatedFat: saturatedFatGramsRange[i] ? ("≤" + saturatedFatGramsRange[i]) : 
                    (i === saturatedFatGramsRange.length ? ">" + saturatedFatGramsRange[i - 1] : ""),
                sugar: totalSugarsGramsRange[i] ? ("≤" + totalSugarsGramsRange[i]) : 
                    (i === totalSugarsGramsRange.length ? ">" + totalSugarsGramsRange[i - 1] : ""),
                points: i,
                id: i,
            };
        });

    const getPoints = (range, nutrition) => range.findIndex(value => nutrition <= value) === -1 ? range.length : range.findIndex(value => nutrition <= value);
    const getPointsFruitVegetableNutLegume = (range, nutrition) => range.findIndex(value => nutrition < value) === -1 ? range.length : range.findIndex(value => nutrition < value);

    $: energyPoints = getPoints(energyRange, energy);
    $: sodiumPoints = getPoints(sodiumRange, sodium);
    $: saturatedFatPoints = getPoints(saturatedFatGramsRange, saturatedFat);
    $: sugarPoints = getPoints(totalSugarsGramsRange, sugar);
    $: baselinePoints = energyPoints + sodiumPoints + saturatedFatPoints + sugarPoints;

    /**
     * Percentage Fruit, Vegetable, Nut or Legume Content (%)
     */
    let percentageFruitVegetableNutLegumeRange = [];

    $: if (Category1D(category) || Category2(category) || Category2D(category) || Category3(category) || Category3D(category)) {
        if (attributes.includes(Attributes.ContainsFruitOrVegetable)) {
            percentageFruitVegetableNutLegumeRange = [25, 43, 52, 63, 67, 80, 90, 100];
        } else if (attributes.includes(Attributes.ContainsNutsOrLegumes)) {
            percentageFruitVegetableNutLegumeRange = [40, 60, 67, 75, 80, 90, 95, 100];
        } else {
            percentageFruitVegetableNutLegumeRange = [];
        }
    } else if (Category1(category)) {
        percentageFruitVegetableNutLegumeRange = [25, 33, 41, 49, 57, 65, 73, 81, 89, 96];
    } else {
        percentageFruitVegetableNutLegumeRange = [];
    }

    $: percentageFruitVegetableNutLegumeData = Array.from({length: 1 + Math.max(percentageFruitVegetableNutLegumeRange.length)}, () => ({}))
        .map((_, i) => {
            return {
                percentageFruitVegetableNutLegume: percentageFruitVegetableNutLegumeRange[i] ? ("<" + percentageFruitVegetableNutLegumeRange[i]) :
                 (i === percentageFruitVegetableNutLegumeRange.length && percentageFruitVegetableNutLegumeRange[i - 1] !== 100 ? "≥" + percentageFruitVegetableNutLegumeRange[i - 1] : "=100"),
                points: i,
                id: i,
            };
        });

    $: percentageFruitVegetableNutLegumePoints = getPointsFruitVegetableNutLegume(percentageFruitVegetableNutLegumeRange, percentageFruitVegetableNutLegumeContent);
  
    /**
     * Fibre (g / 100g)
     */
    let fibreRange = [];

    $: if (Category2(category) || Category2D(category) || Category3(category) || Category3D(category)) {
        fibreRange = [0.9, 1.9, 2.8, 3.7, 4.7, 5.4, 6.3, 7.3, 8.4, 9.7, 11.2, 13, 15, 17.3, 20];
    } else {
        fibreRange = [];
    }

    $: fibreData = Array.from({length: 1 + Math.max(fibreRange.length)}, () => ({}))
        .map((_, i) => {
            return {
                fibre: fibreRange[i] ? ("≤" + fibreRange[i]) :
                 (i === fibreRange.length ? ">" + fibreRange[i - 1] : ""),
                points: i,
                id: i,
            };
        });

    $: fibrePoints = getPoints(fibreRange, fibre);

    /**
     * Protein (g / 100g)
     */
    let proteinRange = [];

    $: if (baselinePoints >= 13 && percentageFruitVegetableNutLegumePoints < 5) {
        proteinRange = [];
    } else if (Category1D(category) || Category2(category) || Category2D(category) || Category3(category) || Category3D(category)) {
        proteinRange = [1.6, 3.1, 4.8, 6.4, 8, 9.6, 11.6, 13.9, 16.7, 20, 24, 28.9, 34.7, 41.6, 50];
    } else {
        proteinRange = [];
    }

    $: proteinData = Array.from({length: 1 + Math.max(proteinRange.length)}, () => ({}))
        .map((_, i) => {
            return {
                protein: proteinRange[i] ? ("≤" + proteinRange[i]) :
                 (i === proteinRange.length ? ">" + proteinRange[i - 1] : ""),
                points: i,
                id: i,
            };
        });

    $: proteinPoints = getPoints(proteinRange, protein);
    $: totalPoints = energyPoints + sodiumPoints + saturatedFatPoints + sugarPoints - (percentageFruitVegetableNutLegumePoints + fibrePoints + proteinPoints);
    
    let pointRange = [];

    $: if (Category1(category)) {
        pointRange = [-Infinity, -Infinity, 0, 1, 3, 5, 7, 9, 11, 12];
    } else if (Category1D(category)) {
        pointRange = [-2, -1, 0, 1, 2, 3, 4, 5, 6, 7];
    } else if (Category2(category)) {
        pointRange = [-11, -7, -2, 2, 6, 11, 15, 20, 24, 25];
    } else if (Category2D(category)) {
        pointRange = [-2, 0, 2, 3, 5, 7, 8, 10, 12, 13];
    } else if (Category3(category)) {
        pointRange = [13, 16, 20, 23, 27, 30, 34, 37, 41, 42];
    } else if (Category3D(category)) {
        pointRange = [24, 26, 28, 30, 31, 33, 35, 37, 39, 40];
    }

    $: ratingData = Array.from({length: pointRange.length}, () => ({}))
        .map((_, i) => {
            return {
                points: pointRange[i] === Number.NEGATIVE_INFINITY ? "" : (
                    i === pointRange.length - 1 ? "≥" + pointRange[i] : "≤" + pointRange[i]
                ),
                rating: 5 - i * 0.5,
                id: i,
            };
        });

</script>
  
<div class="page">
    <div class="title">
        <h1>Health Star Rating Calculator</h1>
        <p>Calculate and explain the <OutboundLink href="http://healthstarrating.gov.au/">Australian/New Zealand health star ratings</OutboundLink> of food.</p>
    </div>
    <div class="calculator">
        <Form on:submit={(e) => {
            e.preventDefault();
        }}>
            <FormGroup>
              <Select id="food-type" labelText="Type of Food* (pick the category most specific to the food)" value="placeholder-item" bind:selected={foodType} on:change={onFormChange}>
                <SelectItem
                  disabled
                  hidden
                  value="placeholder-item"
                  text="Choose an option"
                />
                <SelectItem value="general-food" text="General Food" />
                <SelectItem value="beverage" text="Non-Dairy Beverages" />
                <SelectItem value="dairy-beverage" text="Dairy Beverage" />
                <SelectItem value="dairy-food" text="Dairy Food" />
                <SelectItem value="oil" text="Oils, Fats, and Fat/Oil Spreads" />
                <SelectItem value="cheese" text="Cheese" />
                <SelectItem value="water" text="Plain Water" />
                <SelectItem value="flavoured-water" text="Unsweetened Flavoured Water" />
                <SelectItem value="fruit-and-vegetables" text="Fruit and Vegetables" />
                <SelectItem value="jellies" text="Jellies" />
                <SelectItem value="ice-confection" text="Water-Based Ice Confections" />
              </Select>
            </FormGroup>
            <FormGroup legendText="Required Information">
                <NumberInput label="Energy (kJ) per 100g or 100mL*" hideSteppers bind:value={energy} on:change={onFormChange} />
                <NumberInput label="Sodium (mg) per 100g or 100mL*" hideSteppers bind:value={sodium} on:change={onFormChange} />
                <NumberInput label="Saturated Fat (g) per 100g or 100mL*" hideSteppers bind:value={saturatedFat} on:change={onFormChange} />
                <NumberInput label="Sugar (g) per 100g or 100mL*" hideSteppers bind:value={sugar} on:change={onFormChange} />
            </FormGroup>
            <FormGroup legendText="Optional Information">
                <NumberInput label="Fibre (g) per 100g or 100mL" allowEmpty hideSteppers bind:value={fibre} on:change={onFormChange} />
                <NumberInput label="Protein (g) per 100g or 100mL" allowEmpty hideSteppers bind:value={protein} on:change={onFormChange} />
                <NumberInput label="Percentage Fruit, Vegetable, Nut or Legume Content (%)" allowEmpty step={1} hideSteppers bind:value={percentageFruitVegetableNutLegumeContent} on:change={onFormChange} />
            </FormGroup>
            <FormGroup legendText="Contains Fruit, Vegetable, Nut or Legume">
                <Checkbox id="checkbox-0" labelText="Fruit or Vegetable" bind:checked={containsFruitOrVegetable} on:change={onFormChange} />
                <Checkbox id="checkbox-1" labelText="Nuts or Legumes" bind:checked={containsNutsOrLegumes} on:change={onFormChange} />
            </FormGroup>
          </Form>
    </div>
    <div class="result">
        {#if healthStarRating && healthStarRating > 0 && healthStarRating <= 5}
            <h2>Health Star Rating: </h2>
            <StarRating rating={healthStarRating} style="margin-bottom: 0;" />
            <h2>{healthStarRating}</h2>
        {/if}
    </div>
    <div class="explanation">
        {#if healthStarRating && healthStarRating > 0 && healthStarRating <= 5}
            <h3>Explanation</h3>
            <p>
                Calculation of the Health Star Rating differs based on the category of the food. 
                {#if foodCategoryNotes[foodType] !== ""}
                    {@html foodCategoryNotes[foodType]}
                {/if}
            </p>
            {#if selectItemtoFoodCategoryMap[foodType] === Category.PlainWater}
                <p>
                    Plain water is automatically awarded a 5 star rating.
                </p>
                {:else if selectItemtoFoodCategoryMap[foodType] === Category.UnprocessedFruitAndVegetables}
                <p>
                    Fruit and vegetables are automatically awarded a 5 star rating.
                </p>
            {:else if selectItemtoFoodCategoryMap[foodType] === Category.UnsweetenedFlavouredWater}
                <p>
                    Unsweetened flavoured water is automatically awarded a 5 star rating.
                </p>
            {:else}
                <h5>Baseline Points</h5>
                <p>
                    The food is awarded some "baseline points" to start with, based on its energy, sodium, saturated fat, and sugar contents. 
                    This is basically just a <strong>"bad for you"</strong> score, where the higher these values the higher this baseline score is. 
                </p>
                <p>
                    The baseline points for <strong>{selectItemtoFoodNameMap[foodType]}</strong> are calculated as follows:
                </p>
                <p>
                    Based on the energy, sodium, saturated fat, and sugar contents, the food is awarded baseline points for each of the measures based on the following table:
                </p>
                <Accordion>
                    <AccordionItem title="Baseline Points Table">
                    <DataTable
                        headers={[
                        { key: "energy", value: "Energy (kJ / 100g)" },
                        { key: "sodium", value: "Sodium (mg / 100g)" },
                        { key: "saturatedFat", value: "Saturated Fat (g / 100g)" },
                        { key: "sugar", value: "Sugar (g / 100g)" },
                        { key: "points", value: "Points" },
                        ]}
                        rows={baselineData}
                    />
                    </AccordionItem>
                </Accordion>
                Based on this table, the food has received
                <UnorderedList>
                    <ListItem >{energyPoints} points for the energy of {energy}kJ.</ListItem >
                    <ListItem >{sodiumPoints} points for the sodium of {sodium}mg.</ListItem >
                    <ListItem >{saturatedFatPoints} points for the saturated fat of {saturatedFat}g.</ListItem >
                    <ListItem >{sugarPoints} points for the sugar of {sugar}g.</ListItem >
                </UnorderedList>
                <p>
                    For a total of <strong>{energyPoints + sodiumPoints + saturatedFatPoints + sugarPoints} baseline points</strong>.
                </p>
                <h5>Modifying Points</h5>
                <p>
                    The food is then awarded some "modifying points", based on its protein, fibre, and fruit/vegetable/nut/legume content.
                    This is basically just a <strong>"good for you"</strong> score, where the higher these values the higher this modifying score is.
                </p>
                <p>
                    The modifying points for <strong>{selectItemtoFoodNameMap[foodType]}</strong> are calculated as follows.
                </p>
                {#if percentageFruitVegetableNutLegumeContent !== null && percentageFruitVegetableNutLegumeContent > 0}
                    <p>
                        Based on the % of Fruit/Vegetable/Nut/Legume Content the food is awarded modifying points based on the following table:
                    </p>
                    <Accordion>
                        <AccordionItem title="% of Fruit/Vegetable/Nut/Legume Content Modifying Points Table">
                        <DataTable
                            headers={[
                            { key: "percentageFruitVegetableNutLegume", value: "Percentage Fruit, Vegetable, Nut or Legume Content (%)" },
                            { key: "points", value: "Points" },
                            ]}
                            rows={percentageFruitVegetableNutLegumeData}
                        />
                        </AccordionItem>
                    </Accordion>
                {/if}
                {#if fibre !== null && fibre > 0}
                    {#if Category1(category) || Category1D(category)}
                        <p>
                            <strong>{selectItemtoFoodNameMap[foodType]}</strong> does not get any points for fibre content.
                        </p>
                    {:else}
                        <p>
                            Based on the fibre content the food is awarded modifying points based on the following table:
                        </p>
                        <Accordion>
                            <AccordionItem title="Fibre Modifying Points Table">
                            <DataTable
                                headers={[
                                { key: "fibre", value: "Fibre (g / 100g)" },
                                { key: "points", value: "Points" },
                                ]}
                                rows={fibreData}
                            />
                            </AccordionItem>
                        </Accordion>
                    {/if}
                {/if}
                {#if protein !== null && protein > 0}
                    {#if Category1(category)}
                        <p>
                            <strong>{selectItemtoFoodNameMap[foodType]}</strong> does not get any points for protein content.
                        </p>
                    {:else if baselinePoints >= 13 && percentageFruitVegetableNutLegumePoints < 5}
                        <p>
                            No points will be awarded for protein content as the food has already received 13 or more baseline points and less than 5 modifying points for the % of Fruit/Vegetable/Nut/Legume Content.
                        </p>
                    {:else}
                        <p>
                            Based on the protein content the food is awarded modifying points based on the following table:
                        </p>
                        <Accordion>
                            <AccordionItem title="Protein Modifying Points Table">
                            <DataTable
                                headers={[
                                { key: "protein", value: "Protein (g / 100g)" },
                                { key: "points", value: "Points" },
                                ]}
                                rows={proteinData}
                            />
                            </AccordionItem>
                        </Accordion>
                    {/if}
                {/if}
                {#if (protein == null || protein === 0) && (fibre == null || fibre === 0) && (percentageFruitVegetableNutLegumeContent == null || percentageFruitVegetableNutLegumeContent === 0)}
                    <p>
                        The food did not get any points for protein, fibre, or % of Fruit/Vegetable/Nut/Legume Content, as these values are 0 or have not been input.
                    </p>
                {:else}
                    <p>
                        The food has received:
                    </p>
                    <UnorderedList>
                        <ListItem >{percentageFruitVegetableNutLegumePoints} points for the Percentage Fruit, Vegetable, Nut or Legume Content of {percentageFruitVegetableNutLegumeContent ?? 0}%.</ListItem >
                        <ListItem >{fibrePoints} points for the Fibre of {fibre ?? 0}g.</ListItem >
                        <ListItem >{proteinPoints} points for the Protein of {protein ?? 0}g.</ListItem >
                    </UnorderedList>
                {/if}
                <h5>Total Points</h5>
                <p>
                    The total points for the food is then calculated by subtracting the modifying points from the baseline points:
                </p>
                <p>
                    Baseline Points - Modifying Points = Total Points
                </p>
                <p>
                    {energyPoints + sodiumPoints + saturatedFatPoints + sugarPoints} - ({percentageFruitVegetableNutLegumePoints} + {fibrePoints} + {proteinPoints}) = {energyPoints + sodiumPoints + saturatedFatPoints + sugarPoints - (percentageFruitVegetableNutLegumePoints + fibrePoints + proteinPoints)}
                </p>
                <h5>Health Star Rating</h5>
                <p>
                    The Health Star Rating is then calculated based on the total points (rounded), using the following table:
                </p>
                <DataTable
                    headers={[
                    { key: "points", value: "Total Points" },
                    { key: "rating", value: "Health Star Rating" },
                    ]}
                    rows={ratingData}
                />
                <p>
                    So with the total points (after rounding) of {Math.round(totalPoints)}, the Health Star Rating is {healthStarRating}.
                </p>
            {/if}
        {/if}
    </div>
    <div class="disclaimer">
        <p>
            Disclaimer: This website is not affiliated with the Australian / New Zealand Government and should only be used for educational purposes.
        </p>
        <p>Website by <OutboundLink size="sm" href="https://muhashi.com">muhashi</OutboundLink>.</p>
    </div>
</div>

<style>
    .page {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        height: 100%;
        padding: 1rem 1rem;
        gap: 2rem 0;
    }

    h1, .title > p, .disclaimer > p {
        text-align: center;
    }

    .calculator {
        flex-grow: 1;
    }

    .result {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
        gap: 0 1rem;
    }

    .explanation {
        width: 80%;
        margin-bottom: 4rem;
        display: flex;
        flex-direction: column;
        align-items: flex-start;
        justify-content: center;
        gap: 1rem 0;
    }

    .disclaimer > p {
        opacity: 0.8;
        font-size: small;
    }

</style>
