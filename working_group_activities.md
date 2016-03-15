# Not-Worst Practices for DSE Working Groups (DRAFT)

Bill Howe
March 6, 2016

## Purpose

This document is an initial attempt to capture successful patterns of working group “operations” as reported in the three institutions’ annual reports and the six shared working group reports submitted in December 2015.   This material was motivated by a discussion on the bi-weekly leadership call on February 2/16 in which we noted that although the six working groups across the three institutions did not necessarily need to follow a uniform format, frequency, or participation level for meetings and activities, some working groups would be interested in adopting some standard practices from other working groups’ experience.  Rather than have all working group leads commit to synthesizing material from all nine documents, a suggestion was to extract and assemble some relevant findings in a single place to spur a discussion.

## Approach

We reviewed the nine documents and extracted a list of activities, grouping them (subjectively, but we believe non-controversially) into seven categories:

* Recurring working group meetings
* One-time topic-specific meetings and workshops 
* Consulting/Support/Collaboration activities
* Administrative initiatives
* Digital artifacts (including software)
* Courses and seminar series
* External engagements

For each activity, we indicate the institution (UCB, UW, or NYU) and working group (SWG, Repro, Spaces, ETWG, Careers, EEWG) that led the activity and a very short one-line description of the activity itself.  These activities are grouped and described according to the materials in the annual reports; in those cases where the activity was described as a joint activity with no obvious lead, multiple working groups and/or institutions are mentioned.  

The list is not necessarily exhaustive. Some activities appeared to be described as “sub-activities” and are not included (for example, individual talks in a seminar series).  Research projects were also not included, since the goal of this document is to try and extract best practices for administration and organization of the working groups.

Based on this clustering and informed by the other material in the annual reports, we make some observations and attempt to synthesize a set of best practices.

## Activities by Category

<script type="text/javascript" src="https://www.google.com/jsapi">
</script>
<div id="chart_div" style="width:1250px; height: 450px"><div>
<script>
//Google charts sorting script, best works for stacked bars driven from google sheets, see sample sheet linked in query argument for input format.
// Created : Ethiohelix , http://ethiohelix.blogspot.com/

google.load('visualization', '1', {packages: ['corechart', 'bar']});
google.setOnLoadCallback( function () {
    var Link = 'https://docs.google.com/spreadsheets/d/1YG1jLoPydawyFBDPLyMv0ZLDpUlhfZZlyUv57HAylic/gviz/tq?tqx=out:html&gid=1181212769&headers=1&tq=SELECT+D,count(A)+group+by+D+pivot+B+order+by+D';
    var my_div = "chart_div";

    //set google visualization bar chart options here
    //**********************************************************
    var options = {
        title: 'DSE Activities',
        chartArea: {
            width: '45%',
            height: '70%'
        },
        isStacked: true,
    };
    //***********************************************************
    //pull data from spread sheet (Columns = Labels, Rows = pops)
    // add sheet and range of data at the end like: &range=Sheet1!A1:H8
    var query = new google.visualization.Query(Link);
    query.send(handleQueryResponse);
    function handleQueryResponse(response) {
    var data = response.getDataTable();
    process_data(data,my_div,options);
}  
});


function process_data(data,my_div,options){
    var get_config = chart_config(data,options);
    var options = get_config[0];
    var original_colors = get_config[1];
    var select_counter = 0;
    var chart = new google.visualization.BarChart(document.getElementById(my_div));
    var modifications
    function selectHandler() {
        select_counter +=1;
        //used for sorting selected label
        var selection = chart.getSelection();
        // do for all selections that are valid, if clicking on the actual chart to
        // sort is not desired add && (selection[0].row == null)) to only sort by labels
        if (selection.length > 0)  {
            var selected_column = selection[0].column;
            if (select_counter == 1){
                var selected_label = new google.visualization.DataView(data).getColumnLabel(selected_column);
            }
            else{
                var selected_label = modifications[0].getColumnLabel(selected_column);
            }
            for (var i = 0; i < data.getNumberOfColumns(); i++) {
                var original_label = data.getColumnLabel(i);
                if (selected_label == original_label) {
                    var original_column = i;
                    break
                }
            }
            //sort data and send to view modifier
            data.sort({
                column: original_column,
                desc: true
            });

            modifications = modify_view(original_column, data, original_colors, options);
            //redraw chart
            chart.draw(modifications[0], modifications[1]);
        }
    }
    // 'listener' detects selections made on chart area
    google.visualization.events.addListener(chart, 'select', selectHandler);
    chart.draw(data, options);

}

function modify_view(column, data, original_colors, options) {
    // modifies view after selection to best represent descending sort
    var view = new google.visualization.DataView(data);
    
    //store new sort order in an array
    var new_order = []
    for (var i = 0; i < view.getNumberOfColumns(); i++) {
        if (i == 0) {
            new_order[i] = 0;
        } else if (i == 1) {
            new_order[i] = column;
        } else if ((i == column) && (i != 1)) {
            new_order[i] = 1;
        } else {
            new_order[i] = i;
        }
    }
  

    //Modify series options to use original color scheme
    var modified_options = options;
   
    for (var i = 0; i < view.getNumberOfColumns(); i++) {
        var org_color = original_colors[view.getColumnLabel(new_order[i])];
        if (i != 0) {
            modified_options.series[i - 1] = {color: org_color};
        }
    }
   // set new view and send
    view.setColumns(new_order);
    return [view, modified_options]
}

function chart_config(data,options) {
    //sets and stores parameters for initial chart
    //primary colors to choose from
    var bright_colors = ['#4D4D4D','#5DA5DA','#FAA43A', '#60BD68', '#F17CB0', '#B2912F' ,'#B276B2', '#DECF3F', '#F15854'] 
   
    

    //redefine series colors here and store original config
    options.series = {};
    original_colors = {};
    
    for (var i = 0; i < data.getNumberOfColumns(); i++) {
        if (bright_colors.length > 0){
        var new_color = bright_colors[0];
        // bright_colors[Math.floor(Math.random() * bright_colors.length)];
        bright_colors.splice(bright_colors.indexOf(new_color), 1);
        }
        else{
         var   new_color = getRandomColor();
        }
                     
        if (i != 0) {
            options.series[i - 1] = {color: new_color};
            original_colors[data.getColumnLabel(i)] = new_color;
        }
    }
    return [options, original_colors]
}

function getRandomColor() {
    var letters = '0123456789ABCDEF'.split('');
    var color = '#';
    for (var i = 0; i < 6; i++) {
        color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
}
</script>


### One-time topic-specific meetings and workshops 
UCB SWG: Data Structures for Data Science Workshop (Fernando Perez)
NYU SWG: Introduction to Data Science at IEEE Vis (Claudio Silva)
NYU SWG:  Python for machine learning at conferences (Andreas Mueller)
NYU SWG: Julia meetups (Stefan Karpinski)
UW SWG: Python bootcamp (Jake Vanderplas)
UCB SWG: 200-person Python 3-day course (Josh Bloom, others)
NYU SWG, Spaces: Astro Hackweek (David Hogg, others)
UW SWG: Software Carpentry events
UCB SWG: Software Carpentry events
UW,NYU,UCB Space: Spaces of Sharing Workshop
UW,NYU,UCB Space: Social events
UCB Repro: Reproducibility Workshop
UW ETWG: NSF Workshop on Data Science
UW Careers: Birds-of-a-feather lunches
NYU Careers: day-long meeting to form “Text as Data Association”
NYU Repro: Workshop on indexing software for astronomy and astrophysics 
NYU Methods: Tutorial on Scikit¬Learn 
UW Spaces, SWG: Cloud Day@UW

### Consulting/Support/Collaboration activities
UCB SWG: Office Hours (primarily 50% fellows)
UW SWG: Office Hours (primarily DS staff)
NYU Repro: Office hours on reproducibility (Steeves)
UW SWG: Code Reviews (DS staff and fellows)
UW SWG: Incubator (DS staff)
UW SWG: Data Science for Social Good (DS staff)
NYU SWG: Incubator (staff)
NYU Repro: CERN focused efforts
UCB ETWG: BIDS Collaborative
NYU SWG: Guest lectures
NYU SWG: Capstone projects
UCB SWG: Machine Shop

### Recurring working group meetings
UW SWG: Weekly meetings: Code reviews, reading group, tutorials, guest lectures
NYU Spaces: Joint research meetings with astrophysics group
NYU,UCB,UW Spaces: Bi-weekly cross-institution calls
UW ETWG: Monthly coordination and curriculum meetings
NYU ETWG: Project-focused meetings on cross-institution minors
UCB ETWG: Hacker within
UW ETWG: Python for Sciences
UW Careers: Postdoc lunches
UW Careers: Postdoc entrepreneurial pitch sessions          
NYU Methods: Weekly meeting: Causal inference working group 
UCB EEWG: Reading seminar

### Administrative Initiatives
NYU Careers: new protocol for joint hires between CDS and “domain” departments   UW Careers: 50% “Provost’s Initiative” hires 
UW Careers: Joint postdoc titles between Washington Research Foundation and Moore/Sloan DSE
UW, UCB Careers: “Data Science Fellow” titles awarded to engaged colleagues in various capacities
UW ETWG: “Advanced Data Science Option” at PhD level 
UW ETWG: 6-department masters degree
UCB ETWG: New foundational DS curriculum piloted
UW, NYU, UCB Spaces: New telepresence robots deployed
UW SWG: MetroLab Network and Urban@UW
UW Careers: Affiliates program established to expand “official” network on campus
UW Careers: Dual mentorship protocols established for postdocs, IGERT students, and new faculty
UW Careers, EEWG: Project¬-wide ethnographic analysis   
UCB Careers: Berkeley Career Paths Survey           

### Digital artifacts
All: New DSE web presences
All: Weekly internal communications
NYU EEWG: Data science newsletter
NYU SWG: Software: scikit learn releases
NYU Repro: Software: Reprozip, noWorkflow, VisTrails, ReproMatch
UCB SWG: Software: Numpy Refocus, matplotlib ColorMap, Jupyter
UW SWG: Software: altair, gatspy, astroML, nipy, astropy, mpld3, sqlshare, Myria
UCB Repro: Case Studies
UW Repro: Badges pilot website
NYU Repro: Teaching modules for reproducibility
NYU EEWG: A survey of NYU¬ based scientists designed 
NYU EEWG: Ethnography of data scientists in the DSE and industry 
NYU EEWG: Collaboration network graph visualization and analysis 
UCB Careers: BIDS References Letters      
UCB EEWG: Data Science Studies at Berkeley website

### Courses and seminar series
UCB ETWG, SWG: Foundations of DS course 
UW SWG: Python Seminar 
NYU ETWG: “Data Law and Ethics” (Laura Noren) 
NYU ETWG: “Text as Data” (Arthur Spirling)
NYU Methods, Spaces: Data science lunch seminar series
UW SWG, ETWG: Data Science Seminar
UW ETWG: Community Seminar
UCB Repro: “Reproducible and Collaborative Statistical Data Science”

### External Engagements
UCB SWG: Software Carpentry board membership (Katy Huff)
UW SWG: Software Carpentry board membership (Jake Vanderplas)
NYU, UW, UCB Repro: Engagement with Journals and Conferences on standards
NYU ETWG: Licensing Open Source Software Session 
UW Repro: Research Data Roundtable Discussion with UW Libraries
