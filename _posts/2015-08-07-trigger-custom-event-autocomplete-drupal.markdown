---
layout: post
title: Trigger custom event on autocomplete suggestion selected in Drupal
date: 20:52 07-08-2015
headline: Fine tune autocomplete behavior to suit your every need
taxonomy:
    category: blog
    tag: [database]
---

After failing to configure [Conditional Fields][1] to show fields depending on whether or not a `nodereference` autocomplete field was filled, I came to the conclusion that it would be easier to code a custom behavior by overriding *Drupal's default autocomplete methods*.

<pre><code>(function ($) {
  if (Drupal.jsAC) {
    // Override autocomplete method and add trigger
    // to retrieve the selected node.
    Drupal.jsAC.prototype.hidePopup = function (keycode) {
      // Select item if the right key or mousebutton was pressed.
      if (this.selected && ((keycode && keycode != 46 && keycode != 8 && keycode != 27) || !keycode)) {
        this.select(this.selected);
      }
      // Hide popup.
      var popup = this.popup;
      if (popup) {
        this.popup = null;
        $(popup).fadeOut('fast', function () { $(popup).remove(); });
      }
      this.selected = false;
      $(this.ariaLive).empty();
    };
  }

  $(document).ready(function() {
    $('#autocomplete-field').bind('autocompleteSelect', function(event, node) {
      alert(node.autocompleteValue);
    });
  });
})(jQuery);
</code>
</pre>

Now with a practical use case, let's say we want to **show or hide** a field depending on `nodereference` autocomplete field. But we don't want the field to show if the content of the autocomplete field is not valid (it should contain the node title and the nid between brackets, just like this: `TITLE [nid:NID]`.

<pre><code>(function ($) {
  if (Drupal.jsAC) {
    // Override autocomplete method and add trigger
    // to retrieve the selected node.
    Drupal.jsAC.prototype.hidePopup = function (keycode) {
      // Select item if the right key or mousebutton was pressed.
      if (this.selected && ((keycode && keycode != 46 && keycode != 8 && keycode != 27) || !keycode)) {
        this.select(this.selected);
      }
      // Hide popup.
      var popup = this.popup;
      if (popup) {
        this.popup = null;
        $(popup).fadeOut('fast', function () { $(popup).remove(); });
      }
      this.selected = false;
      $(this.ariaLive).empty();
    };
  }

  $(document).ready(function() {
    // Default to hidden.
    toggleField(false);
    
    $('#autocomplete-field').keyup(function(event) {
        // Check if node is specified in field.
        var state = isValidNode($(this).val());
        toggleField(state);
    });

    $('#autocomplete-field').bind('autocompleteSelect', function(event, node) {
      if (node.autocompleteValue !== '') {
        toggleField(true);
      }
    });

    // Check if node is specified in field.
    function isValidNode(inputText) {
      var pattern = new RegExp(/.*\[nid:[0-9]+\]/g);
      return pattern.test(inputText);
    };

    // Show/Hide other field according to autocomplete
    // field value.
    function toggleField(state) {
      if (state === false) {
        $('#conditional-field').hide();
      } else {
        $('#conditional-field').show();
      }
    }
  });
})(jQuery);
</code>
</pre>

*<u>Note</u>: if you're using <u>Drupal 6</u>, use this code instead:*

<pre><code>(function ($) {
  if (Drupal.jsAC) {
    // Override autocomplete method and add trigger
    // to retrieve the selected node.
    Drupal.jsAC.prototype.hidePopup = function (keycode) {
      // Select item if the right key or mousebutton was pressed.
      if (this.selected && ((keycode && keycode != 46 && keycode != 8 && keycode != 27) || !keycode)) {
        this.input.value = $(this.selected).context.autocompleteValue;
        $(this.input).trigger('autocompleteSelect', $(this.selected));
      }
      // Hide popup
      var popup = this.popup;
      if (popup) {
        this.popup = null;
        $(popup).fadeOut('fast', function() { $(popup).remove(); });
      }
      this.selected = false;
    };
  }

  $(document).ready(function() {
    $('#autocomplete-field').bind('autocompleteSelect', function(event, node) {
      alert(node.autocompleteValue);
    });
  });
})(jQuery);
</code>
</pre>

 [1]: https://www.drupal.org/project/conditional_fields