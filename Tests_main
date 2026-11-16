# Copy this entire block into the main file AFTER all function definitions
# and BEFORE the if __name__ == "__main__": start_server(...) block.
# These tests use their own states, so they do not change the website's data.
from bakery import assert_equal


# Home page: missing information and entry count.
test_state = State("", "", "", [])
assert_equal(index(test_state).content[0], Header("Assembly Desk"))
assert_equal(index(test_state).content[2:6], [
    "Assembly: Untitled", "Assembly number: Not set",
    "Assembly date: Not set", "Part entries: 0",
])

# Assembly setup: current values appear in the form.
test_state = State("Desk", "A-1", "2026-11-15", [])
assert_equal(assembly_setup(test_state).content[2], TextBox("name", "Desk"))
assert_equal(assembly_setup(test_state).content[5], TextBox("number", "A-1"))
assert_equal(assembly_setup(test_state).content[8], TextBox("date", "2026-11-15"))

# Save assembly: trims spaces, saves values, and returns home.
assert_equal(save_assembly(test_state, " New Desk ", " A-2 ", " 2026-12-01 ").content[0],
             Header("Assembly Desk"))
assert_equal(test_state, State("New Desk", "A-2", "2026-12-01", []))

# Empty BOM: helpful message instead of an empty table.
assert_equal(bom(test_state).content[2], "Part entries: 0")
assert_equal(bom(test_state).content[3], "No parts yet. Choose Add Part to start your BOM.")

# Add Part: blank inputs and the correct save action.
assert_equal(add_part(test_state).content[0], Header("Add Part"))
assert_equal(add_part(test_state).content[2], TextBox("part_number", ""))
assert_equal(add_part(test_state).content[6], TextBox("quantity", ""))
assert_equal(add_part(test_state).content[-3],
             SelectBox("scale", ["Not set", "1:1", "1:2", "1:5", "2:1"], "Not set"))
assert_equal(add_part(test_state).content[-2], Button("Save Part", save_part))

# Save Part: adds a record and trims text.
assert_equal(save_part(test_state, " P-1 ", " Bracket ", " 2 ", " Steel ",
                       " 2026-11-01 ", "1:2").content[0], Header("Bill of Materials"))
assert_equal(test_state.parts, [Part("P-1", "Bracket", 2, "Steel", "2026-11-01", "1:2")])

# Populated BOM: row values, count, and buttons identify the selected part.
assert_equal(bom(test_state).content[2], "Part entries: 1")
assert_equal(bom(test_state).content[3], Table([
    ["P-1", "Bracket", 2, "Steel",
     Button("Edit", edit_part, {"part_index": 0}),
     Button("Delete", delete_part, {"part_index": 0})],
], header=["Part number", "Title", "Qty", "Material", "Edit", "Delete"]))

# Edit Part: displays every current field and keeps the selected index.
assert_equal(edit_part(test_state, 0).content, [
    Header("Edit Part"),
    Label("Part number", "part_number"), TextBox("part_number", "P-1"),
    Label("Title", "title"), TextBox("title", "Bracket"),
    Label("Quantity", "quantity"), TextBox("quantity", "2"),
    Label("Material (optional)", "material"), TextBox("material", "Steel"),
    Label("Drawing date (YYYY-MM-DD, optional)", "drawing_date"), TextBox("drawing_date", "2026-11-01"),
    Label("Drawing scale", "scale"), SelectBox("scale", ["Not set", "1:1", "1:2", "1:5", "2:1"], "1:2"),
    Button("Save Changes", update_part, {"part_index": 0}), Button("Cancel", bom),
])

# Update Part: changes only the selected record, even when its number changes.
test_state.parts.append(Part("P-2", "Plate", 1, "", "", "Not set"))
assert_equal(update_part(test_state, 0, "P-NEW", "Updated bracket", "3", "Brass", "", "1:1").content[0],
             Header("Bill of Materials"))
assert_equal(test_state.parts, [
    Part("P-NEW", "Updated bracket", 3, "Brass", "", "1:1"),
    Part("P-2", "Plate", 1, "", "", "Not set"),
])

# Keeping your own part number is allowed, including different capitalization.
assert_equal(update_part(test_state, 0, "p-new", "Updated bracket", "4", "Brass", "", "1:1").content[0],
             Header("Bill of Materials"))
assert_equal(test_state.parts[0].quantity, 4)

# Invalid quantities: both save routes reject them without changing records.
test_saved_parts = test_state.parts.copy()
for test_quantity in ["", " ", "0", "-1", "+1", "1.5", "abc", "1e3", "1 2", "\u00b2", "\u0662"]:
    assert_equal(save_part(test_state, "P-3", "New", test_quantity, "", "", "Not set").content[1],
                 "Please fix: Quantity must be a positive whole number.")
    assert_equal(update_part(test_state, 0, "p-new", "Updated", test_quantity, "", "", "Not set").content[1],
                 "Please fix: Quantity must be a positive whole number.")
    assert_equal(test_state.parts, test_saved_parts)

# Duplicate part numbers: exact, different capitalization, and extra spaces.
for test_number in ["P-2", "p-2", " P-2 "]:
    assert_equal(save_part(test_state, test_number, "New", "1", "", "", "Not set").content[1],
                 "Please fix: That part number is already in the BOM. Edit its quantity if needed.")
    assert_equal(update_part(test_state, 0, test_number, "Updated", "1", "", "", "Not set").content[1],
                 "Please fix: That part number is already in the BOM. Edit its quantity if needed.")
    assert_equal(test_state.parts, test_saved_parts)

# Invalid indexes: cannot edit, add accidentally, or delete another record.
for test_index in [-1, 2, 99]:
    assert_equal(edit_part(test_state, test_index).content[0], Header("Bill of Materials"))
    assert_equal(update_part(test_state, test_index, "P-3", "New", "1", "", "", "Not set").content[0],
                 Header("Bill of Materials"))
    assert_equal(delete_part(test_state, test_index).content[0], Header("Bill of Materials"))
    assert_equal(test_state.parts, test_saved_parts)

# Drawing Review: values, placeholders, and missing details.
assert_equal(drawing_review(test_state).content[0], Header("Drawing Review"))
assert_equal(drawing_review(test_state).content[2], "Part entries: 2")
assert_equal(drawing_review(test_state).content[3], Table([
    ["p-new", "Updated bracket", "Brass", "Not set", "1:1", "Missing: drawing date"],
    ["P-2", "Plate", "Not set", "Not set", "Not set", "Missing: material, drawing date, scale"],
], header=["Part number", "Title", "Material", "Drawing date", "Scale", "Status"]))

# Delete Part: removes only the selected record, then handles the last deletion.
assert_equal(delete_part(test_state, 0).content[0], Header("Bill of Materials"))
assert_equal(test_state.parts, [Part("P-2", "Plate", 1, "", "", "Not set")])
assert_equal(delete_part(test_state, 0).content[3], "No parts yet. Choose Add Part to start your BOM.")
assert_equal(test_state.parts, [])
assert_equal(drawing_review(test_state).content[3],
             "No parts to review yet. Go back to the BOM and choose Add Part.")

# Shared helper: all_digits checks ordinary digits only.
assert_equal(all_digits("123"), True)
assert_equal(all_digits("001"), True)
assert_equal(all_digits("-1"), False)
assert_equal(all_digits("1.5"), False)
assert_equal(all_digits("abc"), False)

# Shared helper: part_error checks required fields, quantity, and duplicates.
test_state = State("", "", "", [Part("P-1", "Bracket", 1, "", "", "Not set")])
assert_equal(part_error(test_state, -1, "P-2", "Plate", "2"), "")
assert_equal(part_error(test_state, -1, "", "Plate", "2"), "Enter both a part number and a title.")
assert_equal(part_error(test_state, -1, "P-2", "", "2"), "Enter both a part number and a title.")
assert_equal(part_error(test_state, -1, "P-2", "Plate", ""), "Quantity must be a positive whole number.")
assert_equal(part_error(test_state, -1, "P-2", "Plate", "0"), "Quantity must be a positive whole number.")
assert_equal(part_error(test_state, -1, "p-1", "Plate", "2"),
             "That part number is already in the BOM. Edit its quantity if needed.")
assert_equal(part_error(test_state, 0, "P-1", "Bracket", "2"), "")

# Shared helper: part_form keeps an error, submitted quantity, and edit target.
assert_equal(part_form(test_state, 0, "P-1", "Bracket", "bad", "", "", "Not set", "Fix quantity.").content[1],
             "Please fix: Fix quantity.")
assert_equal(part_form(test_state, 0, "P-1", "Bracket", "bad", "", "", "Not set", "Fix quantity.").content[7],
             TextBox("quantity", "bad"))
assert_equal(part_form(test_state, 0, "P-1", "Bracket", "bad", "", "", "Not set").content[-2],
             Button("Save Changes", update_part, {"part_index": 0}))

# Shared helper: store_part adds or replaces a record.
assert_equal(store_part(test_state, -1, "P-2", "Plate", "2", "", "", "Not set").content[0],
             Header("Bill of Materials"))
assert_equal(len(test_state.parts), 2)
assert_equal(store_part(test_state, 1, "P-2", "New plate", "3", "", "", "Not set").content[0],
             Header("Bill of Materials"))
assert_equal(test_state.parts[1], Part("P-2", "New plate", 3, "", "", "Not set"))

# Shared helper: drawing_status reports complete and missing information.
assert_equal(drawing_status(Part("P-1", "Bracket", 1, "Steel", "2026-11-01", "1:1")), "Complete")
assert_equal(drawing_status(Part("P-1", "Bracket", 1, "", "2026-11-01", "1:1")), "Missing: material")
assert_equal(drawing_status(Part("P-1", "Bracket", 1, "Steel", "", "1:1")), "Missing: drawing date")
assert_equal(drawing_status(Part("P-1", "Bracket", 1, "Steel", "2026-11-01", "")), "Missing: scale")
assert_equal(drawing_status(Part("P-1", "Bracket", 1, "", "", "Not set")),
             "Missing: material, drawing date, scale")

