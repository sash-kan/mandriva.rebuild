LC_ALL = C
mirror = /mnt/mirrors/mandriva/official/2011
v = 1
ifeq ($(v),0)
	output = &>/dev/null
else
	output =
endif

archs = i586 x86_64
srpms = $(shell cat srpms.list)

cachedir = cache
chrootdir = chroot
#chrootdirs = $(addprefix $(chrootdir)/,$(archs))
#archlabelinchroot = $(foreach a,$(archs),$(chrootdir)/$(a))
archlabelinchroot = $(foreach a,$(archs),$(chrootdir)/tmp/$(a)/)
tmpinchroot = $(addsuffix /tmp,$(chrootdir))
packagesintmpinchroot = $(foreach p,$(srpms),$(foreach a,$(archs),$(tmpinchroot)/$(a)/$(p)))
chroottardir = chroottars
chroottarsarchdir = $(foreach a,$(archs),$(chroottardir)/$(a))
chroottars = $(foreach a,$(archs),$(chroottardir)/$(a).tar)
reportdir = report
reportpackages = $(foreach p,$(addprefix $(reportdir)/,$(srpms)),$(foreach a,$(archs),$(p).$(a)))

srpmsincache = $(addprefix $(cachedir)/,$(srpms))

repos1 = main contrib non-free
repos2 = release updates
repos = $(foreach r1,$repos1,$(foreach r2,$(repos2),$r1/$r2))

percent = %

.SECONDEXPANSION:

#all: srpms.list maketars $(reportdirs) $(chrootdir) $(chrootdirs) $(reportpackages)
all: srpms.list maketars $(chrootdir) $(reportpackages)

maketars: $(chroottardir) $(chroottars)

#$(reportpackages): $$(filter $$(percent)$$(strip $$(foreach a,$$(archs),$$(findstring $$(a),$$@))),$$(chrootdirs))
#$(reportpackages): $$(filter $$(percent)$$(strip $$(foreach a,$$(archs),$$(findstring $$(a),$$@)))/tmp,$$(tmpinchroot))

# $(packagesintmpinchroot):
#$(reportpackages): $$(chrootdir)/tmp/$$(subst .,,$$(suffix $$@))/$$(basename $$(notdir $$@))
# prereq = $(srpmsincache)
$(reportpackages): $(cachedir) $$(cachedir)/$$(basename $$(notdir $$@))
	#echo p=$@ arch=$(strip $(foreach a,$(archs),$(findstring $(a),$@)))

garbage:
	#echo
	#echo $(filter $(percent)$(strip $(foreach a,$(archs),$(findstring $(a),$@)))/tmp,$(tmpinchroot))
	#echo p=$(chrootdir)/tmp/$(basename $(notdir $@)) arch=$(subst .,,$(suffix $@))
	#echo p=$(basename $(notdir $@)) arch=$(subst .,,$(suffix $@))
	#echo p=$@ arch=$(strip $(foreach a,$(archs),$(findstring $(a),$@)))
	@# clear chroot
	@#-sudo rm -rf $(chrootdir)/$(strip $(foreach a,$(archs),$(findstring $(a),$@)))
	@#mkdir $(chrootdir)/$(strip $(foreach a,$(archs),$(findstring $(a),$@)))
	@#sudo tar -xf $(chroottardir)/$(strip $(foreach a,$(archs),$(findstring $(a),$@))).tar -C $(chrootdir)/$(strip $(foreach a,$(archs),$(findstring $(a),$@)))
	#rmdir $(chrootdir)/$(strip $(foreach a,$(archs),$(findstring $(a),$@)))
	#touch $@

# prereq = $(packagesintmpinchroot)
$(srpmsincache): $$(chrootdir)/tmp/$$(word 1,$$(archs))/$$(notdir $$@)
	@echo srpmsincache $@ $(chrootdir)/tmp/*/$(notdir $@)
	cp $(chrootdir)/tmp/*/$(notdir $@) $@

$(tmpinchroot):
	@echo tmpinchroot $@

#$(packagesintmpinchroot): $$(chrootdir)/$$(strip $$(foreach a,$$(archs),$$(findstring $$(a),$$@)))
# prereq = $(archlabelinchroot)
$(packagesintmpinchroot): $$(dir $$@)
	#echo packagesintmpinchroot $@ p=$(notdir $@) arch=$(strip $(foreach a,$(archs),$(findstring $(a),$@)))
	#echo $(dir $@)
	p=$$(urpmq --urpmi-root $(chrootdir) --sources $(basename $(notdir $@))); \
	sudo cp $$p $(chrootdir)/tmp/$(strip $(foreach a,$(archs),$(findstring $(a),$@)))/

#$(archlabelinchroot):
$(archlabelinchroot): $$(chroottardir)/$$(strip $$(foreach a,$$(archs),$$(findstring $$(a),$$@))).tar
	@#echo archlabelinchroot $@ $(subst /,.,$@) $(strip $(foreach a,$(archs),$(findstring $(a),$@)))
	@#echo $(chroottardir)/$(notdir $@).tar
	@#echo $(chroottars)
	sudo tar -xf $^ -C $(chrootdir)
	sudo mkdir $@

#$(chrootdirs): $(chroottars)
#	echo chrootdirs $@ $^
#	mkdir $@

#nall: srpms.list $(cachedir)
#	echo $(srpmsincache)

$(cachedir) $(chrootdir) $(reportdir) $(chroottardir):
	mkdir -p $@

clearall: clearcache clearreports clearchroottars clearchroot

clearcache:
	-rm -rf $(cachedir)

clearreports:
	-rm -rf $(reportdir)

clearchroottars: 
	-sudo rm -rf $(chroottardir)

clearchroot:
	-[ -f .sysmounted ] && sudo umount -lf $(chrootdir)/sys
	-[ -f .procmounted ] && sudo umount -lf $(chrootdir)/proc
	-sudo rm -rf $(chrootdir)

$(chroottarsarchdir): 
	sudo urpmi.addmedia --urpmi-root $@ mainrelease \
	$(mirror)/$(subst $(chroottardir)/,,$@)/media/main/release $(output)
	sudo urpmi.addmedia --urpmi-root $@ contribrelease \
	$(mirror)/$(subst $(chroottardir)/,,$@)/media/contrib/release $(output)
	sudo urpmi.addmedia --urpmi-root $@ non-freerelease \
	$(mirror)/$(subst $(chroottardir)/,,$@)/media/non-free/release $(output)
	sudo urpmi.addmedia --urpmi-root $@ mainupdates \
	$(mirror)/$(subst $(chroottardir)/,,$@)/media/main/updates $(output)
	sudo urpmi.addmedia --urpmi-root $@ contribupdates \
	$(mirror)/$(subst $(chroottardir)/,,$@)/media/contrib/updates $(output)
	sudo urpmi.addmedia --urpmi-root $@ non-freeupdates \
	$(mirror)/$(subst $(chroottardir)/,,$@)/media/non-free/updates $(output)
	#
	sudo urpmi.addmedia --urpmi-root $@ srpmsmainrelease \
	$(mirror)/SRPMS/main/release $(output)
	sudo urpmi.addmedia --urpmi-root $@ srpmscontribrelease \
	$(mirror)/SRPMS/contrib/release $(output)
	sudo urpmi.addmedia --urpmi-root $@ srpmsnon-freerelease \
	$(mirror)/SRPMS/non-free/release $(output)
	sudo urpmi.addmedia --urpmi-root $@ srpmsmainupdates \
	$(mirror)/SRPMS/main/updates $(output)
	sudo urpmi.addmedia --urpmi-root $@ srpmscontribupdates \
	$(mirror)/SRPMS/contrib/updates $(output)
	sudo urpmi.addmedia --urpmi-root $@ srpmsnon-freeupdates \
	$(mirror)/SRPMS/non-free/updates $(output)
	sudo urpmi  --no-suggests --excludedocs --no-verify-rpm --auto --root $@ \
	--urpmi-root $@ shadow-utils rpm tar basesystem-minimal rpm-build \
	rpm-mandriva-setup urpmi rsync bzip2 shadow-utils locales-en $(output)

$(chroottars): $(chroottarsarchdir)
	sudo tar -cf $@ -C $(foreach d,$^,$(findstring $(d),$@)) .

#$(srpms): $$(filter $$(percent)$$@,$(srpmsincache))
#	echo ".src.rpm" $@

