# Rails Foundation 5 and Friendly Printing

## Switching Projects

I’ve decided to take a break for a few days on the [Audio Pila!](https://github.com/asommer70/audiopila-rails) project due to some deadlines that have come blasting up.  So in the mean time I’ve written up a few guides on some different topics related to Rails and web development.

For a work project I’ve needed to create/setup a Printer Friendly view for a form.  Archiving forms on paper shouldn’t be necessary in an ideal world, but some people like hard copy to CYA.

## Adding A Complex Form

For this project we’ll adapt our [Rails Form Project](http://codepen.io/asommer70/post/rails-saving-form-data) from an earlier post and add a more complicated form.  The current test form displays pretty well when printing.  At least in the Print Preview of Chrome…

First off, create a new form in **app/views/forms/complex.html.erb**:

```
<h2>Complex Form</h2>

<%= form_for @input do |f| %>

<div class="row">
  <div class="columns small-12">

    <div class="row">
      <div class="columns small-12">
        <fieldset>
          <legend>
            Vendor Information
          </legend>

          <div class="row">
            <div class="columns small-4">
              <%= text_field_tag 'input[data][vendorname]', @input.data.nil? ? '' : @input.data['vendorname'], placeholder: 'Vendor Name' %>
            </div>

            <div class="columns small-4">
              <%= text_field_tag 'input[data][contactperson]', @input.data.nil? ? '' : @input.data['contactperson'], placeholder: 'Contact Person' %>
            </div>

            <div class="columns small-4">
              <%= text_field_tag 'input[data][address]', @input.data.nil? ? '' : @input.data['street'], placeholder: 'Address' %>
            </div>
          </div>

          <div class="row">
            <div class="columns small-4">
              <%= text_field_tag 'input[data][city]', @input.data.nil? ? '' : @input.data['city'], placeholder: 'City' %>
            </div>

            <div class="columns small-4">
              <%= text_field_tag 'input[data][statezip]', @input.data.nil? ? '' : @input.data['statezip'], placeholder: 'State / Zip' %>
            </div>

            <div class="columns small-4">
              <%= text_field_tag 'input[data][vendorphone]', @input.data.nil? ? '' : @input.data['vendorphone'], placeholder: 'Vendor Phone' %>
            </div>
          </div>
        </fieldset>
      </div>
    </div>


    <div class="row">
      <div class="columns small-12">
        <fieldset>
          <legend>
            Ship To Information
          </legend>

          <div class="row">
            <div class="columns small-6">
              <%= text_field_tag 'input[data][shipToName]', @input.data.nil? ? '' : @input.data['shipToName'], placeholder: 'Name', value: 'The Hoick' %>
            </div>

            <div class="columns small-6">
              <%= text_field_tag 'input[data][shipaddress]', @input.data.nil? ? '' : @input.data['shipaddress'], placeholder: 'Address', value: '10 Elm Street' %>
            </div>
          </div>

          <div class="row">
            <div class="columns small-4">
              <%= text_field_tag 'input[data][shiptocity]', @input.data.nil? ? '' : @input.data['shiptocity'], placeholder: 'City', value: 'Boone' %>
            </div>

            <div class="columns small-4">
              <%= text_field_tag 'input[data][shiptostate]', @input.data.nil? ? '' : @input.data['shiptostate'], placeholder: 'State', value: 'NC' %>
            </div>

            <div class="columns small-4">
              <%= text_field_tag 'input[data][shiptozip]', @input.data.nil? ? '' : @input.data['shiptozip'], placeholder: 'Zip', value: '28607' %>
            </div>
          </div>
        </fieldset>
      </div>
    </div>

    <div class="row">
      <div class="columns small-12">
        <fieldset>
          <legend>
            Requestor Information
          </legend>

          <div class="row">
            <div class="columns small-4">
              <%= text_field_tag 'input[data][requested_by]', @input.data.nil? ? '' : @input.data['requested_by'], placeholder: 'Requested By' %>
            </div>

            <div class="columns small-4">
              <%= text_field_tag 'input[data][date_issued]', @input.data.nil? ? '' : @input.data['date_issued'], placeholder: 'Date Issued' %>
            </div>

            <div class="columns small-4">
              <%= text_field_tag 'input[data][date_required]', @input.data.nil? ? '' : @input.data['date_required'], placeholder: 'Date Required' %>
            </div>
          </div>
        </fieldset>
      </div>
    </div>


      <div class="row">
        <div class="columns small-12">
          <fieldset>
            <legend>
              Items
            </legend>

            <table width="100%">
              <thead>
                <tr bgcolor="#CCCCCC">
                  <th class="text-center">
                    <strong>Line Item #</strong>
                  </th>
                  <th class="text-center">
                    <strong>Qty</strong>
                  </th>
                  <th class="text-center">
                    <strong title="Unit of Measure">U/M</strong>
                  </th>
                  <th class="text-center">
                    <strong>Detailed Description</strong>
                  </th>
                  <th class="text-center">
                    <strong>Cost</strong>
                  </th>
                  <th class="text-center">
                    <strong>Commodity Code</strong>
                    </div>
                  </th class="text-center">
                  <th class="text-center">
                    <strong>TOTAL</strong>
                  </th>
                </tr>
              </thead>
              <tbody class="dynamic" id="items">
                <tr>
                  <td>
                    <input class="dyn-input counter" type="text" id="input[data][item][1]" name="input[data][item][1]" style="width: 50px" />
                  </td>
                  <td>
                    <input class="dyn-input" type="text" id="input[data][quan][1]" name="input[data][quan][1]" style="width: 50px" />
                  </td>
                  <td>
                    <input class="dyn-input" type="text" name="input[data][unit][1]" id="input[data][unit][1]" style="width: 50px" />
                  </td>
                  <td>
                    <textarea class="dyn-input" rows="1" name="input[data][desc][1]" id="input[data][desc][1]" style="width: 300px;" /></textarea>
                  </td>
                  <td>
                    <input class="dyn-input" type="text" id="input[data][cost][1]"  name="input[data][cost][1]" placeholder="$" />
                  </td>
                  <td>
                    <input class="dyn-input" type="text" id="input[data][disc][1]" name="input[data][disc][1]" />
                  </td>
                  <td>
                    <input class="dyn-input row_total" type="text" id="input[data][total][1]" name="input[data][total][1]" />
                  </td>
                </tr>
              </tbody>
            </table>

            <input type="hidden" name="input[data][items_rc]" id="input[data][items_rc]" data-table="items" />

            <button class="button tiny secondary add-row" data-table="items" id="items-add">
                <i class="fi-plus"></i> &nbsp; Add Row
            </button>
          </fieldset>
        </div>
      </div>

      <div class="row">
        <div class="columns small-4">
          <%= submit_tag 'Save Form', class: 'button small save-form' %>
        </div>
      </div>

      <%= hidden_field_tag 'input[form_name]', @input.form_name %>
<% end %>
```

There’s a dynamic table at the bottom which we’ll add some JavaScript to enable.

## CoffeeScript For Dynamic Tables

There’s an [article](http://codepen.io/asommer70/post/dynamic-table-rows-with-coffeescript) out there about using CoffeeScript to create dynamic tables full of input fields, and now we’re going to expand on that a little.  

Create a new file **app/assets/javascripts/inputs.coffee** with:

```
ready_tables = ->
  #
  # Handle add-row button clicks.
  #
  $('.add-row').on 'click', (e) ->
    e.preventDefault()
    table_body = $(e.target).data().table
    if table_body
      add_row(table_body)

  #
  # Click add-row the number of
  #
  $.get(window.location + '.json')
    .then (input) ->
      # Click the .add-row button the appropriate number of times.
      for k, v of input.data
        if k.substring(k.length - 3, k.length) == '_rc'
          for i in [0...parseInt(v)] by 1
            $('.add-row').click()

      # Loop through the input.data a second time.
      for key, value of input.data
        # Set the dynamic table fields.
        for k, v of value
          sel = "#input\\\\[data\\\\]\\\\[#{key}\\\\]\\\\[#{k}\\\\]"
          console.log('sel:', sel)
          #$input = $("##{key}\\[#{k}\\]")
          $input = $("#input\\[data\\]\\[#{key}\\]\\[#{k}\\]")
          console.log('$input:', $input)
          $input.val(v)


add_row = (table_body_element) ->
# Get some variables for the tbody and the row to clone.
  $tbody = $('#' + table_body_element)
  $rows = $($tbody.children('tr'))
  $cloner = $rows.eq(0)
  count = $rows.length

  # Clone the row and get an array of the inputs.
  $new_row = $cloner.clone()
  inputs = $new_row.find('.dyn-input')

  # Change the name and id for each input.
  $.each(inputs, (i, v) ->
    $input = $(v)

    # Find the label for input and adjust it.
    $label = $new_row.find("label[for='#{$input.attr('id')}']")
    $label.attr( {'for': $input.attr('id').substring(0, $input.attr('id').lastIndexOf('[')) + "[#{count + 1}]"} )

    $input.attr({
      'name': $input.attr('name').substring(0, $input.attr('name').lastIndexOf('[')) + "[#{count + 1}]",
      'id': $input.attr('id').substring(0, $input.attr('id').lastIndexOf('[')) + "[#{count + 1}]"
    })

    # Remove values and checks.
    $input.val('')
    checked = $input.prop('checked')
    if checked
      $input.prop('checked', false)
  )

  # Add the new row to the tbody.
  $tbody.append($new_row)

# Fire the ready function on load and refresh.
$(document).ready(ready_tables)
$(document).on('page:load', ready_tables)
```

Notice the changes from the original:

* Changed the **$(‘#add-row)** on click handler to a class selector **$(‘.add-row)**.
* Changed the **replace** method call with **substring** to adjust the new row’s label elements, input id attributes, and input name attributes.
* Also, added code to get the JSON for an Input and fill the values of dynamic tables.

## Update the Show Method

To enable the JSON for an Input the **show** method in **app/controllers/inpts_controller.rb** needs to be updated to render JSON when appropriate:

```
  def show
    @input = Input.find(params[:id])

    unless request.format == 'application/json'
      render template: "forms/#{@input.form_name}"
    end
  end
```

## Printer Friendly Views/CSS

With all that work done, time to adjust the CSS for better printing.  As you can see from this screenshot:

![](img/bad_print.png)

The text fields above the table aren’t in a 3 column grid, there’s no shading of odd rows in the table, and the navigation is visible at the top.  Let’s take care of those things via CSS (or SASS rather).

To make things look good with styles, we need to enable the **print** device in our stylesheets in the **app/views/layouts/application.html.erb** at the end of the **head** element add:

```
    <%= stylesheet_link_tag "application", media: "print" %>
```

There is another **stylesheet_link_tag** entry above, but adding the **media: “print”** argument enables the print styles.

Next, create a new file **app/assets/stylesheets/print.scss**:

```
@media print {
  .sub-nav, .add-row, .save-form {
    display: none;
  }

  div.columns {
    display: table !important;
    position: relative;
    padding-left: 0.9375rem;
    padding-right: 0.9375rem;
    float: left;
  }

  .small-4 {
    max-width: 193pt;
  }

  .small-6 {
    max-width: 290pt;
  }

  fieldset {
    margin: 5pt;
  }

  thead {
    background-color: #cccccc !important;
    -webkit-print-color-adjust: exact;
  }

  .dynamic tr:nth-child(even) {
    background-color: #f9f9f9 !important;
    -webkit-print-color-adjust: exact;
  }

  @page {
    size: auto !important;   /* auto is the initial value */
    margin: .5pt !important;
    margin-top: 14pt !important;
    margin-bottom: 14pt !important;
    page-break-after: avoid !important;
  }

  label {
    display: block;
  }
}
```

From this [post](http://stackoverflow.com/questions/23372405/nth-child-not-displaying-in-print-media) I guess you can’t apply background colors to print jobs.  The work around for [Webkit](https://www.webkit.org/) browsers is to use the **-webkit-print-color-adjust: exact;** style.  At least that worked for me.

Hit Ctrl+p (Command+p on a Mac) to bring up the Print Preview window in Chrome and you’ll see the new styles applied.

![](img/good_print.png)

## Conclusion

It’s a lot more work to support printing… at least that was my opinion.  It’s a world of screens, and I’m sure glad I don’t have to deal with printing websites on a day to day basis.

Also, our little Forms app is much more functional now that we can have dynamic tables in our forms and have the data visible later.

Party On!