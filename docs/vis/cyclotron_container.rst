Widget Container
================

The Widget Container widget allows to position other widgets within itself as if it were a page inside the page, i.e., with its own grid layout and theme. Its configuration is similar to that of a regular Page.

Layout Configuration
-------------

The following table illustrates the layout configuration properties available.

================================ ==================== ========================= ==================
Property                         JSON Key             Default                   Description
================================ ==================== ========================= ==================
Grid Columns                     gridColumns                                    Total number of horizonal grid squares available in the grid. The grid squares will be scaled to fit the container. If omitted, the number of columns is calculated dynamically.
Grid Rows                        gridRows                                       Total number of vertical grid squares available in the grid. The grid squares will be scaled vertically to fit the container. If omitted, the grid squares will be literally square, i.e., the height and width will be the same. When omitted, the widgets may not fill the entire container, or they may scroll vertically. Use this property to make widgets scale vertically to fit the container.
Grid Container Height Adjustment gridHeightAdjustment ``0``                     Adjustment (in pixels) to the height of the container when calculating the grid layout. If the value is positive, the container height will have the adjustment added to it (making each row taller). If it is negative, the container height will be reduced (making each row shorter).
Grid Container Width Adjustment  gridWidthAdjustment  ``0``                     Adjustment (in pixels) to the width of the container when calculating the grid layout. If the value is positive, the container width will have the adjustment added to it (making each column wider). If it is negative, the container width will be reduced (making each column skinnier).
Gutter                           gutter               ``10``                    Space (in pixels) between widgets positioned in the grid
Border Width                     borderWidth                                    Pixel width of the border around each widget. Can be set to 0 to remove the border. If omitted, the theme default will be used.
Margin                           margin               ``10``                    Empty margin width (in pixels) around the outer edge of the container. Can be set to 0 to remove the margin.
Scrolling Enabled                scrolling            ``true``                  Enables vertical scrolling of the container to display content longer than the current size
================================ ==================== ========================= ==================
