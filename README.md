import {
  Button,
  Dialog,
  DialogActions,
  DialogContent,
  Grid,
  MenuItem,
  Tooltip,
} from "@mui/material";
import React, { useMemo, useState } from "react";
import { useDispatch, useSelector } from "react-redux";
import { updateSkillsInStore } from "../../redux/slices/avatarSlice";
import CustomButton from "../../utils/CustomButton";
import CustomTextField from "../../utils/CustomTextField";
import ModalHeader from "../../utils/ModalHeader";
import SkillTable from "../SkillTable/SkillTable";
import { useTranslation } from "react-i18next";
// import debounce from "lodash.debounce";

const EditProfileModal = ({ avatar, onClose }) => {
  const dispatch = useDispatch();
  const avatars = useSelector((state) => state?.avatars?.avatars);
  const [searchType, setSearchType] = useState("skill");
  const [searchQuery, setSearchQuery] = useState("");
  const [skills, setSkills] = useState(
    avatars.find((a) => a.id === avatar.id)?.skills || []
  );

  // const debouncedSetSearchQuery = useCallback(
  //   debounce((query) => {
  //     setSearchQuery(query);
  //   }, 300),
  //   []
  // );

  // const handleSearchChange = (e) => {
  //   debouncedSetSearchQuery(e.target.value);
  // };

  const { t } = useTranslation();

  const filteredSkills = useMemo(() => {
    return skills.filter((skill) =>
      searchType === "skill"
        ? skill?.name?.toLowerCase()?.includes(searchQuery?.toLowerCase())
        : skill?.rating?.toLowerCase()?.includes(searchQuery?.toLowerCase())
    );
  }, [skills, searchQuery, searchType]);

  const handleAddSkill = () => {
    const newSkillObject = { name: "", rating: "", editing: true, isNew: true };
    setSkills([...skills, newSkillObject]);
  };

  const handleApply = () => {
    dispatch(updateSkillsInStore({ avatarId: avatar.id, newSkills: skills }));
    onClose();
  };

  const handleCancel = () => {
    setSkills(avatars.find((a) => a.id === avatar.id)?.skills || []);
    onClose();
  };

  const handleSkillChange = (index, field, value) => {
    const updatedSkills = skills.map((skill, i) =>
      i === index ? { ...skill, [field]: value } : skill
    );
    setSkills(updatedSkills);
  };

  const handleSkillBlur = (index, field, value) => {
    if (!value) {
      const updatedSkills = skills.map((skill, i) =>
        i === index ? { ...skill, [field]: value, error: true } : skill
      );
      setSkills(updatedSkills);
    }
  };

  const handleSkillDelete = (index) => {
    const updatedSkills = skills.filter((_, i) => i !== index);
    setSkills(updatedSkills);
  };

  const handleSkillEditToggle = (index) => {
    const updatedSkills = skills.map((skill, i) =>
      i === index ? { ...skill, editing: !skill.editing } : skill
    );
    setSkills(updatedSkills);
  };

  const hasEmptyFields = useMemo(() => {
    return skills.some((skill) => !skill?.name || !skill?.rating);
  }, [skills]);

  return (
    <Dialog
      open
      onClose={onClose}
      fullWidth
      maxWidth="lg"
      PaperProps={{ style: { height: "80vh", borderRadius: "20px" } }}
      data-testid="edit-profile-modal"
    >
      <ModalHeader title={t("Edit Profile")} onClose={onClose} />
      <DialogContent>
        <Grid container spacing={2}>
          <Grid item xs={12} sm={4}>
            <img
              src={avatar.image}
              alt={avatar.name}
              style={{ width: "100%", height: "100%" }}
            />
          </Grid>
          <Grid item xs={12} sm={8}>
            <Grid container spacing={2}>
              <Grid item xs={6}>
                <CustomTextField
                  fullWidth
                  label={t("Search Skill")}
                  variant="outlined"
                  margin="normal"
                  value={searchQuery}
                  onChange={(e) => setSearchQuery(e.target.value)}
                  // onChange={handleSearchChange}
                  inputProps={{ "data-testid": "search-skill-input" }}
                />
              </Grid>
              <Grid item xs={6}>
                <CustomTextField
                  fullWidth
                  select
                  label={t("Select Skill/Rating")}
                  variant="outlined"
                  margin="normal"
                  value={searchType}
                  onChange={(e) => setSearchType(e.target.value)}
                  inputProps={{ "data-testid": "select-skill-rating" }}
                >
                  <MenuItem value="skill">{t("Skill")}</MenuItem>
                  <MenuItem value="rating">{t("Rating")}</MenuItem>
                </CustomTextField>
              </Grid>
            </Grid>
            <SkillTable
              skills={filteredSkills}
              onSkillChange={handleSkillChange}
              onSkillBlur={handleSkillBlur}
              onSkillDelete={handleSkillDelete}
              onSkillEditToggle={handleSkillEditToggle}
              setSkills={setSkills}
            />
            <div style={{ marginTop: "20px", textAlign: "center" }}>
              <Button
                variant="contained"
                onClick={handleAddSkill}
                sx={{
                  borderRadius: 8,
                  textTransform: "none",
                  fontWeight: "bold",
                  backgroundColor: "#000",
                  color: "#fff",
                  boxShadow: "0 3px 5px rgba(0, 0, 0, 0.2)",
                  "&:hover": {
                    backgroundColor: "#333",
                  },
                }}
                style={{ marginTop: "10px", width: "100%" }}
                data-testid="add-skill-button"
              >
                {t("Add Skill")}
              </Button>
            </div>
          </Grid>
        </Grid>
      </DialogContent>
      <DialogActions>
        <CustomButton text={t("Cancel")} onClick={handleCancel} />
        {hasEmptyFields && (
          <Tooltip
            title={t("Please fill in all fields before applying")}
            placement="top"
          >
            <div>
              <CustomButton
                text={t("Apply")}
                onClick={handleApply}
                disabled={hasEmptyFields}
              />
            </div>
          </Tooltip>
        )}
        {!hasEmptyFields && (
          <CustomButton
            text={t("Apply")}
            onClick={handleApply}
            disabled={hasEmptyFields}
          />
        )}
      </DialogActions>
    </Dialog>
  );
};

export default React.memo(EditProfileModal);

import React, { useState } from "react";
import {
  IconButton,
  Menu,
  MenuItem,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableRow,
  Tooltip,
} from "@mui/material";
import { Info, MoreVert } from "@mui/icons-material";
import CustomTextField from "../../utils/CustomTextField";
import { useTranslation } from "react-i18next";

const SkillTable = ({
  skills,
  onSkillChange,
  onSkillBlur,
  onSkillDelete,
  onSkillEditToggle,
  setSkills,
}) => {
  const { t } = useTranslation();
  const [anchorEl, setAnchorEl] = useState(null);
  const [currentSkillIndex, setCurrentSkillIndex] = useState(null);

  const handleMenuOpen = (event, index) => {
    setAnchorEl(event.currentTarget);
    setCurrentSkillIndex(index);
  };

  const handleMenuClose = () => {
    setAnchorEl(null);
    setCurrentSkillIndex(null);
  };

  const handleSkillAction = (actionType) => {
    const currentSkill = skills[currentSkillIndex];
    if (!currentSkill.name || !currentSkill.rating) {
      alert("Skill name and rating cannot be empty.");
      return;
    }
    onSkillEditToggle(currentSkillIndex);
    if (actionType === "apply" || actionType === "update") {
      const updatedSkills = [...skills];
      updatedSkills[currentSkillIndex] = {
        ...currentSkill,
        isNew: false,
        editing: false,
      };
      setSkills(updatedSkills);
    }
    handleMenuClose();
  };

  const handleDeleteSkillClick = () => {
    onSkillDelete(currentSkillIndex);
    handleMenuClose();
  };

  return (
    <Paper style={{ marginTop: "10px" }}>
      <Table data-testid="skill-table">
        <TableHead>
          <TableRow>
            <TableCell>{t("Skill")}</TableCell>
            <TableCell>{t("Rating")}</TableCell>
            <TableCell>{t("Actions")}</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {skills?.length > 0 ? (
            skills.map((skill, index) => (
              <TableRow key={index}>
                <TableCell>
                  <CustomTextField
                    value={skill.name}
                    onChange={(e) =>
                      onSkillChange(index, "name", e.target.value)
                    }
                    onBlur={(e) => onSkillBlur(index, "name", e.target.value)}
                    error={!skill.name}
                    helperText={!skill.name && t("Skill name is required")}
                    inputProps={{
                      "data-testid": `skill-name-${index}`,
                    }}
                    disabled={!skill.editing}
                    aria-label="Name"
                  />
                  {skill.editing && !skill.name && (
                    <Tooltip
                      title={t("Skill name is required")}
                      placement="top"
                    >
                      <Info color="error" />
                    </Tooltip>
                  )}
                </TableCell>
                <TableCell>
                  <CustomTextField
                    value={skill.rating}
                    onChange={(e) =>
                      onSkillChange(index, "rating", e.target.value)
                    }
                    onBlur={(e) => onSkillBlur(index, "rating", e.target.value)}
                    error={!skill.rating}
                    helperText={!skill.rating && t("Rating is required")}
                    inputProps={{
                      "data-testid": `skill-rating-${index}`,
                    }}
                    disabled={!skill.editing}
                    aria-label="Rating"
                  />
                  {skill.editing && !skill.rating && (
                    <Tooltip title={t("Rating is required")} placement="top">
                      <Info color="error" />
                    </Tooltip>
                  )}
                </TableCell>
                <TableCell>
                  <IconButton
                    onClick={(e) => handleMenuOpen(e, index)}
                    aria-controls={`skill-menu-${index}`}
                    aria-haspopup="true"
                    data-testid={`skill-actions-button-${index}`}
                  >
                    <MoreVert />
                  </IconButton>
                  <Menu
                    id={`skill-menu-${index}`}
                    anchorEl={anchorEl}
                    keepMounted
                    open={Boolean(anchorEl) && currentSkillIndex === index}
                    onClose={handleMenuClose}
                  >
                    <MenuItem
                      onClick={() =>
                        handleSkillAction(
                          skills[currentSkillIndex]?.isNew
                            ? "apply"
                            : skills[currentSkillIndex]?.editing
                            ? "update"
                            : "edit"
                        )
                      }
                    >
                      {skills[currentSkillIndex]?.isNew
                        ? t("Apply")
                        : skills[currentSkillIndex]?.editing
                        ? t("Update")
                        : t("Edit")}
                    </MenuItem>
                    <MenuItem onClick={handleDeleteSkillClick}>
                      {t("Delete")}
                    </MenuItem>
                  </Menu>
                </TableCell>
              </TableRow>
            ))
          ) : (
            <TableRow>
              <TableCell colSpan={3} align="center">
                {t("No skills available")}
              </TableCell>
            </TableRow>
          )}
        </TableBody>
      </Table>
    </Paper>
  );
};

export default React.memo(SkillTable);

import { createAsyncThunk, createSlice } from "@reduxjs/toolkit";
import { fetchAvatars } from "../../api/avatarApi";

export const fetchAvatarsApi = createAsyncThunk(
  "avatars/fetchAvatars",
  async () => {
    const response = await fetchAvatars();
    return response;
  }
);

const initialState = {
  avatars: [],
  loading: false,
  error: null,
};

const avatarSlice = createSlice({
  name: "avatars",
  initialState,
  reducers: {
    addSkill: (state, action) => {
      const { avatarId, newSkill } = action.payload;
      const avatar = state.avatars.find((a) => a.id === avatarId);
      if (avatar) {
        avatar.skills.push({ ...newSkill, editing: false }); // Set editing to false by default
      }
    },
    deleteSkill: (state, action) => {
      const { avatarId, skillIndex } = action.payload;
      const avatar = state.avatars.find((a) => a.id === avatarId);
      if (avatar) {
        avatar.skills.splice(skillIndex, 1);
      }
    },
    updateSkill: (state, action) => {
      const { avatarId, skills } = action.payload;
      const avatar = state.avatars.find((a) => a.id === avatarId);
      if (avatar) {
        avatar.skills = skills.map((skill) => ({ ...skill, editing: false })); // Reset editing to false for all skills
      }
    },
    updateSkillsInStore: (state, action) => {
      const { avatarId, newSkills } = action.payload;
      const avatar = state.avatars.find((a) => a.id === avatarId);
      if (avatar) {
        const existingSkills = avatar?.skills;
        const updatedSkills = newSkills?.reduce((acc, newSkill) => {
          const existingIndex = existingSkills.findIndex(
            (skill) =>
              skill.name === newSkill?.name &&
              skill?.rating === newSkill?.rating
          );
          if (existingIndex === -1) {
            acc.push({ ...newSkill, editing: false }); // Set editing to false for new skills
          } else {
            acc.push(existingSkills[existingIndex]);
          }
          return acc;
        }, []);
        avatar.skills = updatedSkills;
      }
    },
    toggleSkillEditing: (state, action) => {
      const { avatarId, skillIndex, editing } = action.payload;
      const avatar = state.avatars.find((a) => a.id === avatarId);
      if (avatar) {
        avatar.skills[skillIndex].editing = editing;
      }
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchAvatarsApi.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchAvatarsApi.fulfilled, (state, action) => {
        state.loading = false;
        state.avatars = action?.payload;
      })
      .addCase(fetchAvatarsApi.rejected, (state, action) => {
        state.loading = false;
        state.error = action?.error?.message;
      });
  },
});

export const {
  addSkill,
  updateSkill,
  deleteSkill,
  updateSkillsInStore,
  toggleSkillEditing,
} = avatarSlice.actions;

export default avatarSlice.reducer;
