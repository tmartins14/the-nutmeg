# =============================================================================
# pieces/01-world-cup-48/Makefile
# Run from the piece root: cd pieces/01-world-cup-48
# =============================================================================

PYTHON = python
SCRIPTS = scripts

.PHONY: normalize

normalize:
	$(PYTHON) $(SCRIPTS)/normalize_team_names.py

team-name-fix:
	$(PYTHON) $(SCRIPTS)/team_name_fix.py

elo:
	$(PYTHON) $(SCRIPTS)/calculate_elo.py