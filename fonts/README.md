# Fonts

Arcadia's figure fonts, static instances from Google Fonts, licensed under the
SIL Open Font License (see the `OFL-*.txt` files). They are loaded from here by
`analysis.elm.apply_figure_style()` rather than from the system, so the figures
render the same on every machine without installing anything.

Only the regular weights are included, matching the published figures, which
were rendered with regular-weight axis labels. Adding a Medium instance would
make the labels bolder, as Arcadia's style specifies.
