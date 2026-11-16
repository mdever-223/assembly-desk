"""Assembly Desk: organize one assembly and review its drawing details."""

from dataclasses import dataclass
from drafter import set_website_framed
from drafter import LineBreak

from drafter import (
    Button, Header, Label, Page, SelectBox, Table, TextBox,
    hide_debug_information, route, set_site_information, set_website_title,
    start_server,
)


@dataclass
class Part:
    """Store the BOM and drawing details for one kind of part."""

    part_number: str
    title: str
    quantity: int
    material: str
    drawing_date: str
    scale: str


@dataclass
class State:
    """Keep the current assembly and its part records across pages."""

    assembly_name: str
    assembly_number: str
    assembly_date: str
    parts: list[Part]


@route
def index(state: State) -> Page:
    """Show the current assembly and links to setup and the BOM."""
    return Page(state, [
        Header("Assembly Desk"),
        "Organize the parts and drawing details for one assembly.",
        "Assembly: " + (state.assembly_name or "Untitled"),
        "Assembly number: " + (state.assembly_number or "Not set"),
        "Assembly date: " + (state.assembly_date or "Not set"),
        "Part entries: " + str(len(state.parts)),
        "Assembly details are kept while you use the site. Reloading starts over.",
        Button("Assembly Setup", assembly_setup),
        Button("View BOM", bom),
    ])


@route
def assembly_setup(state: State) -> Page:
    """Show a form for the assembly name, number, and date."""
    return Page(state, [
        Header("Assembly Setup"),
        Label("Assembly name", "name"), TextBox("name", state.assembly_name),LineBreak(),
        Label("Assembly number", "number"), TextBox("number", state.assembly_number),LineBreak(),
        Label("Assembly date (YYYY-MM-DD)", "date"), TextBox("date", state.assembly_date),LineBreak(),
        Button("Save Assembly", save_assembly),
        Button("Cancel", index),
    ])


@route
def save_assembly(state: State, name: str, number: str, date: str) -> Page:
    """Save the three assembly fields and return to the dashboard."""
    state.assembly_name = name.strip()
    state.assembly_number = number.strip()
    state.assembly_date = date.strip()
    return index(state)


@route
def bom(state: State) -> Page:
    """Build a BOM table and attach edit and delete actions to each row."""
    rows = []
    part_index = 0
    for part in state.parts:
        rows.append([
            part.part_number, part.title, part.quantity, part.material or "Not set",
            Button("Edit", edit_part, {"part_index": part_index}),
            Button("Delete", delete_part, {"part_index": part_index}),
        ])
        part_index+=1
    if not rows:
        listing = "No parts yet. Choose Add Part to start your BOM."
    else:
        listing = Table(rows, header=["Part number", "Title", "Qty", "Material", "Edit", "Delete"])
    return Page(state, [
        Header("Bill of Materials"),
        "Assembly: " + (state.assembly_name or "Untitled"),
        "Part entries: " + str(len(state.parts)),
        listing,
        Button("Add Part", add_part),
        Button("Drawing Review", drawing_review),
        Button("Home", index),
    ])


def part_form(
    state: State, part_index: int, part_number: str, title: str, quantity: str,
    material: str, drawing_date: str, scale: str, error: str = "",
) -> Page:
    """Build the shared add/edit form and preserve entries after an error."""
    if part_index == -1:
        heading = "Add Part"
        submit = Button("Save Part", save_part)
    else:
        heading = "Edit Part"
        # The button passes the row position separately from the editable fields.
        submit = Button("Save Changes", update_part, {"part_index": part_index})
    content = [Header(heading)]
    if error:
        content.append("Please fix: " + error)
    content += [
        Label("Part number", "part_number"), TextBox("part_number", part_number),
        Label("Title", "title"), TextBox("title", title),
        Label("Quantity", "quantity"), TextBox("quantity", quantity),
        Label("Material (optional)", "material"), TextBox("material", material),
        Label("Drawing date (YYYY-MM-DD, optional)", "drawing_date"), TextBox("drawing_date", drawing_date),
        Label("Drawing scale", "scale"), SelectBox("scale", ["Not set", "1:1", "1:2", "1:5", "2:1"], scale),
        submit,
        Button("Cancel", bom),
    ]
    return Page(state, content)


@route
def add_part(state: State) -> Page:
    """Show an empty form for a new part."""
    return part_form(state, -1, "", "", "", "", "", "Not set")

def part_error(state: State, part_index: int, part_number: str, title: str, quantity: str) -> str:
    """Return the first problem that prevents saving a part, or empty text."""
    if not part_number or not title:
        return "Enter both a part number and a title."
    if not quantity or not all_digits(quantity):
        return "Quantity must be a positive whole number."
    if int(quantity) <= 0:
        return "Quantity must be a positive whole number."
    other_index = 0
    for part in state.parts:
        if other_index != part_index and part.part_number.lower() == part_number.lower():
            return "That part number is already in the BOM. Edit its quantity if needed."
        other_index += 1
    return "" 


def all_digits(quantity: str) -> bool:
    """Check for ordinary digits before converting the quantity to an integer."""
    for character in quantity:
        if character not in "0123456789":
            return False
    return True


def store_part(
    state: State, part_index: int, part_number: str, title: str, quantity: str,
    material: str, drawing_date: str, scale: str,
) -> Page:
    """Validate the form and add or replace a record on success."""
    part_number = part_number.strip()
    title = title.strip()
    quantity = quantity.strip()
    error = part_error(state, part_index, part_number, title, quantity)
    if error:
        return part_form(state, part_index, part_number, title, quantity, material, drawing_date, scale, error)
    part = Part(part_number, title, int(quantity), material.strip(), drawing_date.strip(), scale)
    if part_index == -1:
        state.parts.append(part)
    else:
        state.parts[part_index] = part
    return bom(state)


@route
def save_part(
    state: State, part_number: str, title: str, quantity: str,
    material: str, drawing_date: str, scale: str,
) -> Page:
    """Save a new part if its required fields are valid."""
    return store_part(state, -1, part_number, title, quantity, material, drawing_date, scale)

@route
def edit_part(state: State, part_index: int) -> Page:
    """Show the selected part's current values in the shared form."""
    if not 0 <= part_index < len(state.parts):
        return bom(state)
    part = state.parts[part_index]
    return part_form(
        state, part_index, part.part_number, part.title, str(part.quantity),
        part.material, part.drawing_date, part.scale,
    )


@route
def update_part(
    state: State, part_index: int, part_number: str, title: str, quantity: str,
    material: str, drawing_date: str, scale: str,
) -> Page:
    """Validate changes and save them to the selected part."""
    if not 0 <= part_index < len(state.parts):
        return bom(state)
    return store_part(
        state, part_index, part_number, title, quantity,
        material, drawing_date, scale,
    )


@route
def delete_part(state: State, part_index: int) -> Page:
    """Remove the selected part and show the updated BOM."""
    if 0 <= part_index < len(state.parts):
        state.parts.pop(part_index)
    return bom(state)


def drawing_status(part: Part) -> str:
    """List the optional drawing details still missing from a record."""
    missing = []
    if not part.material:
        missing.append("material")
    if not part.drawing_date:
        missing.append("drawing date")
    if not part.scale or part.scale == "Not set":
        missing.append("scale")
    if missing:
        return "Missing: " + ", ".join(missing)
    return "Complete"


@route
def drawing_review(state: State) -> Page:
    """Show drawing details and flag missing fields for each part."""
    rows = []
    for part in state.parts:
        rows.append([
            part.part_number, part.title, part.material or "Not set",
            part.drawing_date or "Not set", part.scale or "Not set", drawing_status(part),
        ])
    if not rows:
        summary = "No parts to review yet. Go back to the BOM and choose Add Part."
    else:
        summary = Table(rows, header=["Part number", "Title", "Material", "Drawing date", "Scale", "Status"])
    return Page(state, [
        Header("Drawing Review"),
        "Assembly: " + (state.assembly_name or "Untitled"),
        "Part entries: " + str(len(state.parts)),
        summary,
        Button("Back to BOM", bom),
    ])


set_website_title("Assembly Desk")
set_site_information(
    author="Matthew",
    description="Organize one assembly, build a BOM, and review missing drawing details.",
    sources="Drafter documentation and ChatGPT assistance (include the full conversation in the repository).",
    planning="See the brainstorming and design worksheets in the project repository.",
    links=[],
)
hide_debug_information()
set_website_framed(False)

if __name__ == "__main__":
    start_server(State("", "", "", []))
