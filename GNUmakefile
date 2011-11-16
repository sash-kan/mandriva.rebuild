LC_ALL = C
mirror = /mnt/mirrors/mandriva/official/2011
v = 0

archs = i586 x86_64
srpms = $(shell cat srpms.list)

cachedir = cache
chrootdir = chroot
chroottardir = chroottars
chroottarsarchdir = $(foreach a,$(archs),$(chroottardir)/$(a))
packagesintmpinchroottar = $(foreach p,$(srpms),$(foreach d,$(chroottarsarchdir),$(d)/tmp/$(p)))
chroottars = $(foreach a,$(archs),$(chroottardir)/$(a).tar)
reportdir = report
reportpackages = $(foreach p,$(addprefix $(reportdir)/,$(srpms)),$(foreach a,$(archs),$(p).$(a)))

srpmsincache = $(addprefix $(cachedir)/,$(srpms))

repos1 = main contrib non-free
repos2 = release updates
repos = $(foreach r1,$repos1,$(foreach r2,$(repos2),$r1/$r2))

percent = %
archat = $(strip $(foreach a,$(archs),$(findstring $(a),$@)))
ifeq ($(v),0)
	output = &>/dev/null
else
	output =
endif

.SECONDEXPANSION:

all: srpms.list maketars $(reportpackages)

maketars: $(chroottardir) $(chroottars)

# prereq = cache $(srpmsincache) report
$(reportpackages): $(cachedir) $$(cachedir)/$$(basename $$(notdir $$@))

# prereq = chroottars/i586/tmp packagesintmpinchroottar
$(srpmsincache): $(chroottardir)/$(word 1,$(archs)) $$(chroottardir)/$$(word 1,$$(archs))/tmp/$$(notdir $$@)
	cp $$(ls $(chroottardir)/*/tmp/$(notdir $@)) $@

$(packagesintmpinchroottar):
	p=$$(urpmq --urpmi-root $(chroottardir)/$(archat) \
	--sources $(basename $(notdir $@))); \
	sudo cp $$p $(chroottardir)/$(archat)/tmp/

$(cachedir) $(reportdir) $(chroottardir):
	mkdir -p $@

clearall: clearcache clearreports clearchroottars clearchroot

clearcache:
	-rm -rf $(cachedir)

clearreports:
	-rm -rf $(reportdir)

clearchroottars: 
	-sudo rm -rf $(chroottardir)

clearchroot:
	-[ -f .sysmounted ] && sudo umount -lf $(chrootdir)/sys; :
	-[ -f .procmounted ] && sudo umount -lf $(chrootdir)/proc; :
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

